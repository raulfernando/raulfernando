---
layout: post
title:  "Data Transfer Object em Ruby"
date:   2019-06-18 09:16:29 -0300
author: Raul Fernando
categories: Ruby Design Pattern
---
Data Transfer Object é um design pattern usado para encapsular dados em um objeto. O benefício dos DTOs é converter os dados em um objeto e reduzir informações desnecessárias. Além disso o DTO torna o código muito fácil de manter e testar.

Supondo que estamos escrevendo um código para obter o endereço de um fornecedor.

Vejamos o exemplo abaixo:
```ruby
response = Faraday.get('http://example-domain.com/provider')
```

A resposta é assim:

```javascript
{
  "provider": {
    "name": "Fornecedor",
    "address": {
      "street": "Av. Paulista",
      "number": "1612",
      "complement": "-",
      "zipcode": "02460100",
      "district": "Centro",
      "city": "São Paulo",
      "state": "SP"
    }
  }
}
```

para acessarmos os attributos da resposta precisaremos fazer algo como

```ruby
result = JSON.parse(response)
result['provider']['address']['street']
# => 'AV Paulista'
result['provider']['address']['city']
# => 'São Paulo'
```

Como você pode ver, o código acima não é realmente eficiente, em caso de
legibilidade e escalabilidade. Além disso, o que aconteceria quando alguns
campos na estrutura da hierarquia precisassem ser alterados?

## Implementando o pattern de DTO

Primeiro, crie uma nova classe chamada DTO::Address. Essa classe é responsável
por traduzir os dados retornados da API para um objeto.

```ruby
class Address
  attr_reader :street, :number, :complement, :zipcode, :district, :city, :state

  def initialize(data)
    @street     = data['street'],
    @number     = data['number'],
    @complement = data['complement'],
    @zipcode    = data['zipcode'],
    @district   = data['district'],
    @city       = data['city'],
    @state      = data['state']
  end

  def full_address
    "#{street}, #{number} - #{district} - #{city} - #{state}, #{zipcode}"
  end
end
```

Com `DTO::Address`, poderíamos instanciar um novo objeto passando os dados respondidos da API.

```ruby
result = JSON.parse(result)
address = DTO::Address.new(result['provider']['address'])
```

Todas as informações respondidas da API são encapsuladas dentro do DTO::Address.
Apenas informações necessárias com alguma lógica de domínio adicional são incluídas no objeto por exemplo um metodo para retornar o endereço completo formatado do fornecedor.

```ruby
address.full_address
# => 'Av. Paulista, 1612 - Centro - São Paulo - SP - 02460100'
```

O código agora é muito mais limpo, sustentável, testável e escalável.
