---
layout: post
title: "Utilizando componentes do ActiveModel fora do Rails"
author: carlos
category:
- Ruby
permalink: componentes-active-model-fora-do-rails
meta_description: "Utilizando componentes do ActiveModel fora do Rails"
browser_title: "Utilizando componentes do ActiveModel fora do Rails"
---

Aqui no Moip temos o hábito de escrever apps ruby puras sem qualquer framework, como o Rails, Sinatra ou Grape, às vezes por que não temos necessidade das funcionalidades destes frameworks às vezes só por que é divertido :stuck_out_tongue:

Num destes projetos, um web worker responsável por gerar relatórios, houve a necessidade de serializarmos hashes retornados por buscas do Elasticsearch em hashes no formato das linhas de um relatório específico. Para solucionar esse problema optamos por utilizar o <a href="https://github.com/rails-api/active_model_serializers" target="_blank">ActiveModel::Serializer</a> o único porém é que nosso app não utiliza Rails então a dúvida era como integrar este componente sem o uso do Rails...

Para quem não conhece o ActiveModel::Serializer é uma gem responsável por realizar serialização de objetos em JSON de forma simples e altamente customizável. Geralmente é utilizado em app's Rails ou Rails API, porém nada nos impede de utilizá-lo em app's que não façam uso do Rails sendo que sua integração em uma app não Rails é bem simples de ser feita e neste post darei um breve exemplo de como consegui-lo :smiley:

Primeiro, basta adicionar estas gem's ao seu GEMFILE:

{% highlight ruby linenos %}
gem 'activesupport'
gem 'active_model_serializers'
{% endhighlight %}

Para o exemplo usaremos um model chamado Pessoa, com os seguintes atributos:

{% highlight ruby linenos %}
class Pessoa
  # Torna esse model serializável pelo ActiveModel::Serialization
  include ActiveModel::Serialization

  attr_accessor :idade, :altura, :peso

  # Retorna qual classe é responsável pode serializar este model
  def active_model_serializer
    PessoaSerializer
  end
end
{% endhighlight %}

Por fim devemos criar um serializer para este model na mão mesmo:

{% highlight ruby linenos %}
class PessoaSerializer < ActiveModel::Serializer
  attributes :idade, :altura, :peso, :imc

  def imc
    altura_em_metros = object.altura.to_f / 100
    imc = object.peso.to_f / (altura_em_metros * altura_em_metros)
    imc.round 2
  end
end
{% endhighlight %}

No exemplo acima, através do método attributes realizamos um mapeamento um-para-um, ou seja, o serializer irá procurar por um atributo altura, por exemplo, no model serializado e irá criar um nó com este mesmo nome. Também é possível criar nós customizados, com nomes ou valores que não existem no model serializado, o método imc ilustra esta possibilidade.

Feito isso serializar objetos da classe Pessoa é bem simples:

{% highlight ruby linenos %}
carlos = Pessoa.new
carlos.idade = 26
carlos.altura = 180
carlos.peso = 95
serializer = PessoaSerializer.new carlos, root: false

puts "Como JSON => #{serializer.to_json}" #=> Como JSON => {"idade":26,"altura":180,"peso":95,"imc":29.32}
puts "Como Hash => #{serializer.serializable_hash}" #=> Como Hash => {:idade=>26, :altura=>180, :peso=>95, :imc=>29.32}
{% endhighlight %}

Como podemos notar pelo exemplo acima, é muito fácil incluir o ActiveModel::Serializer no seu projeto e aproveitar seus benefícios com pouco código, caso queria dar uma olhada no exemplo completo o código do post está disponível neste repositório do github: <a href="http://github.com/moip/active-model-components-outside-rails" target="_blank">active-model-components-outside-rails</a> :+1: