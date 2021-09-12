---
layout: post
title: Testando o Método Create de um Controller com Nested Attributes
---

Como testar o método create de um controller que utiliza nested_attributes

-----

<p>
  Estava eu essa semana tentando terminar os scripts de teste de um projeto. Tudo estava dando certo, terminei de cobrir todos os testes dos <i>models</i> e comecei a trabalhar nos <i>controllers</i>, até que me deparei com o teste do método <i>create</i> de um <i>controller</i> que utiliza <i>nested_attributes</i> para cadastro de um <i>model</i> e suas associações. A estrutura dos <i>models</i> e <i>controller</i> é semelhante a essa:
</p>

``` ruby
# app/models/author.rb
class Author < ActiveRecord::Base
  has_many :posts, dependent: :destroy

  accepts_nested_attributes_for :posts, allow_destroy: true

  ...
end
```

``` ruby
# app/models/post.rb
class Post < ActiveRecord::Base
  belongs_to :author

  ...
end
```

``` ruby
# app/controllers/authors_controller.rb
class AuthorsController < ApplicationController
  ...

  def create
    @author = Author.new author_params

    if @author.save
      flash[:success] = 'Autor cadastrado com sucesso!'
      redirect_to @author
    else
      render 'new'
    end
  end

  ...

  private

  def author_params
    params.require(:author).permit(:name, posts_attributes: [
                                            :title, :content,
                                            :id, :_destroy
                                          ])
  end

  ...
end
```

<p>
  Eu utilizo FactoryGirl para facilitar meus testes, a principio pesquisei por algum método mágico dessa gema que convertesse a Hash dos atributos exatamente para o aceito pelo parâmetro do controller (com <i>nested_attributes</i>). Passei longos 30 minutos e não achei nada, então resolvi fazer a seguinte gambiarra no script de teste:
</p>

``` ruby
require 'rails_helper'

RSpec.describe AuthorsController, type: :controller do
  ...

  describe 'POST #create' do
    context 'with valid attributes' do
      let!(author_params) do
        FactoryGirl.attributes_for(:author)
                   .merge(posts_attributes: [FactoryGirl.attributes_for(:post)])
      end

      it 'creates a new author' do
        expect {
          post :create, author: author_params
        }.to change { Author.count }.by(1)
      end

      it 'redirects to :show view' do
        post :create, author: author_params
        expect(response).to redirect_to(Author.last)
      end
    end

    ...
  end

  ...
end
```

<p>
  Acredito que este não seja a forma mais correta de testar esse tipo de situação, mas pra mim deu certo.
</p>
