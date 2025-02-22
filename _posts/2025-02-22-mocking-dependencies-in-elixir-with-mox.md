---
layout: post
title:  "Mockando Dependências em Elixir com Mox"
date:   2025-02-22 00:27:29 -0300
author: Raul Fernando
categories: Elixir
---

# Mockando Dependências em Elixir com Mox

No desenvolvimento de aplicações Elixir, frequentemente nos deparamos com a necessidade de testar módulos que dependem de serviços externos, como APIs de terceiros ou bancos de dados. Para garantir que nossos testes sejam rápidos e confiáveis, podemos usar mocks. O Mox, uma biblioteca oficial da comunidade Elixir, nos permite criar mocks de maneira segura e eficaz.

Neste artigo, vamos aprender a configurar e utilizar o Mox para mockar dependências em testes.

## Configurando o Mox

Para começar, adicione o Mox ao seu projeto adicionando a dependência no seu `mix.exs`:

```elixir
# mix.exs
defp deps do
  [
    {:mox, "~> 1.0", only: :test}
  ]
end
```

Em seguida, rode `mix deps.get` para instalar a dependência.

## Criando um Comportamento (Behaviour)

O Mox funciona com `behaviours`, que são contratos definidos para módulos em Elixir. Vamos criar um comportamento para um cliente HTTP fictício:

```elixir
# lib/my_app/http_client.ex
defmodule MyApp.HttpClient do
  @callback get(String.t()) :: {:ok, String.t()} | {:error, String.t()}
end
```

Esse comportamento define que qualquer módulo que o implemente deve ter uma função `get/1` que retorna um `{:ok, String.t()}` em caso de sucesso ou `{:error, String.t()}` em caso de falha.

## Criando o Mock

Agora, criamos um mock baseado no comportamento acima:

```elixir
# test/support/mocks.exs
Mox.defmock(MyApp.HttpClientMock, for: MyApp.HttpClient)
```

Para garantir que o mock seja carregado corretamente em nossos testes, adicionamos essa linha ao `test/test_helper.exs`:

```elixir
Code.require_file("support/mocks.exs", __DIR__)
```

## Usando o Mock nos Testes

Agora podemos utilizar o mock nos testes. Vamos supor que temos um módulo que depende do `HttpClient`:

```elixir
# lib/my_app/service.ex
defmodule MyApp.Service do
  @http_client Application.compile_env(:my_app, :http_client, MyApp.HttpClient)

  def fetch_data(url) do
    case @http_client.get(url) do
      {:ok, body} -> {:ok, "Processed: " <> body}
      {:error, reason} -> {:error, reason}
    end
  end
end
```

Podemos então configurar o nosso teste para usar o mock:

```elixir
# test/my_app/service_test.exs
defmodule MyApp.ServiceTest do
  use ExUnit.Case, async: true
  import Mox

  setup :verify_on_exit!

  test "fetch_data/1 returns processed data when HTTP request succeeds" do
    MyApp.HttpClientMock
    |> expect(:get, fn _url -> {:ok, "mock response"} end)

    Application.put_env(:my_app, :http_client, MyApp.HttpClientMock)

    assert MyApp.Service.fetch_data("http://example.com") == {:ok, "Processed: mock response"}
  end

  test "fetch_data/1 returns error when HTTP request fails" do
    MyApp.HttpClientMock
    |> expect(:get, fn _url -> {:error, "failed request"} end)

    Application.put_env(:my_app, :http_client, MyApp.HttpClientMock)

    assert MyApp.Service.fetch_data("http://example.com") == {:error, "failed request"}
  end
end
```

## Conclusão

O Mox é uma ferramenta poderosa para mockar dependências em testes Elixir, garantindo que possamos testar nossos módulos de forma isolada e previsível. Ele segue boas práticas, como garantir que todas as expectativas definidas sejam chamadas e que as implementações sejam validadas com `behaviours`.

Com essa abordagem, seus testes se tornam mais rápidos, confiáveis e fáceis de manter. 🚀

