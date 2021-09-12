---
layout: post
title: Capturando erro de parse com JSON inválido
---

Como capturar erro de parse com JSON inválido utilizando middleware do Rails

-----

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
O problema é que a requisição nem chega a acessar a *action* do *controller* já que o erro
***ActionDispatch::Http::Parameters::ParseError*** é disparado no ***Action Dispatch*** do ***Rails***.

## Solução

A ideia por trás da solução é criar um middleware para capturar o erro e
retornar uma resposta para o usuário.

Um middleware necessita de dois métodos, ***initialize*** e ***call***. Os middlewares
são executados todas as vezes que uma requisição é recebida pela aplicação
da seguinte forma: Primeiro o middleware é instanciado com a própria aplicação
e em seguida é chamado o método ***call***, que por final chamará o middleware seguinte.

Nosso middleware só precisa executar o método ***call*** e capturar o erro
disparado ao tentar realizar o parse do JSON com erro.

Criaremos nossa classe do midleware em `app/middleware/catch_json_parse_errors.rb`:

``` ruby
# app/middleware/catch_json_parse_errors.rb

class CatchJsonParseErrors
  FAILSAFE_RESPONSE = [
    400, { 'Content-Type' => 'application/json' },
    [
      {
        status: 400,
        message: 'Parse error on payload body.'
      }.to_json
    ]
  ].freeze

  def initialize app
    @app = app
  end

  def call env
    @app.call(env)
  rescue ActionDispatch::Http::Parameters::ParseError => exception
    is_content_type_json?(env) ? FAILSAFE_RESPONSE : raise(exception)
  end

  private

  def is_content_type_json? env
    env['CONTENT_TYPE'] =~ /application\/json/
  end
end
```

Para que nossa classe seja adicionada na inicialzação da aplicação devemos adicionar
a seguinte linha em `config/application.rb`:

``` ruby
# config/application.rb

module MyApplication
  class Application < Rails::Application
    # ...

    config.middleware.use CatchJsonParseErrors

    # ...
  end
end
```
