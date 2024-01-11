# Padrão da cadeia de responsabilidade

## Problema
Quando um sistema exige uma série de etapas de processamento, ele geralmente enfrenta o desafio de lidar com diversas solicitações com diferentes necessidades de processamento. Cada solicitação pode exigir um manipulador diferente, o que dificulta a integração e o gerenciamento perfeitos do processo de manipulação.

## Solução
Nesses casos, o padrão Chain of Responsibility se mostra benéfico. Ele permite que o sistema organize os manipuladores em uma cadeia, cada um capaz de processar solicitações específicas. O remetente inicia uma solicitação sem especificar seu destinatário final, permitindo um tratamento dinâmico e flexível. Se um manipulador não puder processar a solicitação, ele a passará para o próximo na cadeia. Isso promove o desacoplamento, o dimensionamento e a adaptabilidade no processamento de solicitações.

## Exemplo
Vamos supor que temos uma loja onde os clientes compram vários itens. Queremos aplicar prêmios com base no histórico de compras dos clientes.
Suponha que tenhamos as seguintes regras para diferentes prêmios 
  - Se o valor da compra for superior a 1.500 centavos, adicione a recompensa "próxima compra grátis".
  - Se for a segunda compra nos últimos trinta dias, adicione a recompensa "twenty percent off next order"-reward (vinte por cento de desconto no próximo pedido)
  - Se a compra for feita em 4 de maio, "item temático de Star Wars adicionado à entrega"-recompensa

Podemos ter classes diferentes para lidar com a regra

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

Vamos criar uma classe base RewardRule
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

Vamos criar uma classe RewardManager final que gerencie todas as regras

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

Agora podemos usar um objeto `RewardManager` para aplicar regras de correspondência e, se todas forem correspondentes, aplicar a última regra.

```ruby
# CUSTOMER_PURCHASES manterá os dados das compras do cliente, que terão customer_id, purchase_amount_cents e created_at
reward_manager = RewardManager.new
results = CUSTOMER_PURCHASES.map do |purchase|
  reward_manager.apply_reward(purchase[:customer_id], purchase)
end

puts results
```