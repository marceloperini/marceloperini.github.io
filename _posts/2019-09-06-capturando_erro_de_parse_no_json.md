---
title: "Capturando erro de parse com JSON inválido"
layout: post
date: 2019-09-06 22:00:00
image: /assets/images/post1.jpg
headerImage: true
tag:
  - rails
  - json
category: blog
author: marcelo
description: "Como capturar erro de parse com JSON inválido utilizando middleware do Rails"
---

## O problema

Imagine a seguinte situação:

> # "
> Você possui uma aplicação com uma Api externa disponibilizada para os seus
> clientes, obviamente você não consegue controlar o conteúdo e nem o formato
> que serão enviados para os *endpoints*. Em um belo dia você se depara com um
> número crescente de erros 500 acontecendo devido ao *parse* do conteúdo passado
> no *payload*.
> # "

O erro é parecido com esse:

```
Error occurred while parsing request parameters.
Contents:

{
  invalid JSON
}
```

## Entendendo o ocorrido

Esse erro é disparado [nesta linha](https://github.com/rails/rails/blob/98a57aa5f610bc66af31af409c72173cdeeb3c9e/actionpack/lib/action_dispatch/http/parameters.rb#L114)
e ocorre todas as vezes em que não é possível realizar o *parse* do conteúdo enviado por uma requisição.
O problema é que a requisição nem chega a acessar a *action* do *endpoint* já que erro ocorre no ***Action Dispatch*** do ***Rails***.

## Solução
