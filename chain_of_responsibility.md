# Chain of Responsibility Pattern

## Problem
When a system requires a series of processing steps, it often faces the challenge of handling diverse requests with varying processing needs. Each request may demand a different handler, making it hard to seamlessly integrate and manage the handling process.

## Solution
In such cases, the Chain of Responsibility pattern proves beneficial. It allows the system to organize handlers into a chain, each capable of processing specific requests. The sender initiates a request without specifying its ultimate receiver, enabling dynamic and flexible handling. If a handler can't process the request, it passes it to the next in the chain. This promotes decoupling, scalability, and adaptability in request processing.

## Example
Let's assume we have store where customers make purchase of various items. we want to apply rewards based on the past history of customer purchases.
Suppose we have below rules for different rewards - 
  - If the purchase amount higher than 1500 cents, add reward "next purchase free"-reward
  - If second purchase in the past thirty days, add reward "twenty percent off next order"-reward
  - If Purchase on 4th May, "Star Wars themed item added to delivery"-reward

we can have different classes for handling the the rule

```ruby
class PurchaseAmountRule < RewardRule
  def apply(customer, purchase)
    if purchase[:purchase_amount_cents] > 1500
      'Next purchase free'
    else
      @successor&.apply(customer, purchase)
    end
  end
end

class PurchaseOnMay4thRule < RewardRule
  def apply(customer, purchase)
    if purchase[:created_at].day == 4 && purchase[:created_at].month == 5
      'Star Wars themed item added to delivery'
    else
      @successor&.apply(customer, purchase)
    end
  end
end

class SecondPurchaseRule < RewardRule
  def apply(customer, purchase)
    customer_purchases = CUSTOMER_PURCHASES.select { |p| p[:customer_id] == customer }
    purchase_index = customer_purchases.index { |p| p == purchase }
    days = calculate_difference_in_days(purchase, customer_purchases[purchase_index - 1])

    if customer_purchases.length > 1 && purchase_index.positive? && days <= 30
      'Twenty percent off next order'
    else
      @successor&.apply(customer, purchase)
    end
  end

  private

  def calculate_difference_in_days(purchase, last_purchase)
    return 0 if last_purchase.nil?

    difference_in_seconds = (purchase[:created_at] - last_purchase[:created_at]).to_i
    difference_in_seconds / (24 * 60 * 60)
  end

end
```

Let's create a base class RewardRule
```ruby
class RewardRule
  attr_reader :successor

  def initialize(successor = nil)
    @successor = successor
  end

  def apply(customer, purchase)
    raise NotImplementedError, 'Subclasses must implement the apply method'
  end
end
```

Let's create a final RewardManager class which manage all rules

```ruby
require_relative 'purchase_amount_rule'
require_relative 'second_purchase_rule'
require_relative 'purchase_on_may_4th_rule'

class RewardManager
  def initialize
    @rule_chain = build_rule_chain
  end

  def apply_reward(customer, purchase)
    @rule_chain.apply(customer, purchase) || 'No reward'
  end

  private

  def build_rule_chain
    # we can either define a constant array or yaml config file
    rule_config = YAML.load_file('rule_configuration.yaml')
    rule_chain = nil

    rule_config['rules'].each do |rule_class_name|
      rule_class = Object.const_get(rule_class_name)
      rule_chain = rule_class.new(rule_chain)
    end

    rule_chain
  end
end
```

```yml
rules:
  - PurchaseAmountRule
  - SecondPurchaseRule
  - PurchaseOnMay4thRule
```
Now we can use a `RewardManager` object to apply matching rules and if all are matching then apply the last rule.

```ruby
# CUSTOMER_PURCHASES will hold data of customer's purchases which will have customer_id, purchase_amount_cents and created_at
reward_manager = RewardManager.new
results = CUSTOMER_PURCHASES.map do |purchase|
  reward_manager.apply_reward(purchase[:customer_id], purchase)
end

puts results
```
