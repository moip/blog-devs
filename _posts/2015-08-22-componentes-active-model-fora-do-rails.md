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

Num destes projetos, um web worker responsável por gerar relatórios, houve a necessidade de serializarmos hashes retornados por buscas do Elasticsearch em hashes no formato das linhas de um relatório específico. Para solucionar esse problema optamos por utilizar o [ActiveModel::Serializer](https://github.com/rails-api/active_model_serializers) o único porém é que nosso app não utiliza Rails então a dúvida era como integrar este componente sem o uso do Rails...

Posso dizer que não foi trivial resolver isso, já que freqüentemente pode ser bem complicado encontrar gem's com documentação explicando como utilizá-las em apps não Rails, porém a parte boa é que no caso do ActiveModel::Serializer é perfeitamente possível integrá-lo à um app sem que se faça necessário o uso de Rails e neste post darei um breve exemplo de como consegui-lo :smiley: