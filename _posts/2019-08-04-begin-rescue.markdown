---
layout: post
title:  "Como usar as palavras-chave Begin & Rescue"
date:   2019-08-04 09:16:29 -0300
author: Raul Fernando
categories: Ruby Exception
---

Neste post irei falar sobre `begin` e `rescue` palavras chaves utilizadas no ruby para lidar com erros. Por exemplo, ao tentar acessar um arquivo inexistente ou tentar fazer uma divisão por zero será levantado um erro (uma exception) quando isso acontece o ruby não falhará imediatamente dando ao desenvolvedor a chance  lidar com eles caso ocorram, isso geralmente é chamado de "exception handling".

O ruby nos fornece duas palavras-chaves para lidarmos com as exceptions geradas.
Essas palavras-chave são `begin` e `rescue` vejamos como utiliza-las

Dentro do `begin` tem o código que você vai executar e que pode levantar uma exception. Exemplo:

```ruby
begin
  IO.sysopen('/foo/bar')
rescue
  # ...
end
```

No código acima estamos tentando abrir um arquivo, uma exception será levantada caso o arquivo não possa ser aberto, sendo assim usamos a palavra-chave `rescue` para resgatar o erro e adicionarmos o código que lidará com a exception veja o exemplo abaixo:

```ruby
begin
  IO.sysopen('/foo/bar')
rescue
  puts "Arquivo não encontrado."
end
```

Além disso o `rescue` pode levar um argumento opcional. Este argumento é a classe da exception que você deseja resgatar, por exemplo o `IO` pode levantar um erro de arquivo não encontrado que será `Errno::ENOENT` Ou `Errno::EACCES` por um erro de permissão. Dito isso existe uma forma para o desenvolvedor manipular varias exceptions no mesmo bloco de (`begin` / `rescue`) como mostrado no exemplo abaixo:

```ruby
begin
  IO.sysopen('/foo/bar')
rescue Errno::ENOENT
  puts "Arquivo não encontrado."
rescue Errno::EACCES
  puts "Você não possui permissão para abrir este arquivo."
end
```

E se você quiser que o mesmo código seja executado para varias exceptions você poderá faze-lo da seguinte maneira:

```ruby
begin
  IO.sysopen('/foo/bar')
rescue Errno::ENOENT, Errno::EACCES
  puts "Houve um erro ao abrir o arquivo.."
end
```

Uma coisa legal é que nem sempre você precisa usar a palavra-chave `begin`, quando dentro de um método ou bloco a mesma pode ser omitida. Veja o exemplo abaixo:

```ruby
def get_null_device
  IO.sysopen('/foo/bar')
rescue Errno::ENOENT
  puts "O arquivo solicitado não pode ser aberto."
end
```

```ruby
["a.txt", "b.txt", "c.txt"].map  do |f|
  IO.sysopen(f)
rescue  Errno::ENOENT
  puts  "Can't open IO device: #{f}."
end
```

A própria definição do método em si faz o trabalho de `begin`, então você pode omiti-lo.

**Lembrando que é uma boa pratica você lidar com exceptions  especificas para evitar esconder erros de você mesmo.**
