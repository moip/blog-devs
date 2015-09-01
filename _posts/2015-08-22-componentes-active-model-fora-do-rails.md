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

Posso dizer que não foi trivial resolver isso, já que freqüentemente pode ser bem complicado encontrar gem's com documentação explicando como utilizá-las em apps não Rails, porém a parte boa é que no caso do ActiveModel::Serializer é perfeitamente possível integrá-lo à um app sem que se faça necessário o uso de Rails e neste post darei um breve exemplo de como consegui-lo :smiley:

Primeiro, basta adicionar estas gem ao seu GEMFILE:

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

Por fim, como estamos numa app não Rails, devemos criar um serializer para este model na mão mesmo:

{% highlight ruby linenos %}
class PessoaSerializer < ActiveModel::Serializer
  attributes :idade, :altura, :peso, :imc

  def imc
    altura_em_metros = object.altura.to_f / 100
    object.peso.to_f / (altura_em_metros * altura_em_metros)
  end
end
{% endhighlight %}

No exemplo acima, através do método attributes realizamos um mapeamento um-para-um, ou seja, o serializer irá procurar por um atributo altura, por exemplo, no model serializado e irá criar um nó com este mesmo nome. Também é possível criar nós customizados, com nomes ou valores que não existem no model serializado, o método imc ilustra esta possibilidade.