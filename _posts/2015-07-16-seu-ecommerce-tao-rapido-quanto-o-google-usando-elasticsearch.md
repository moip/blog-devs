---
layout: post
title: "Seu e-commerce tão rápido quanto o Google usando ElasticSearch"
category: 
- elastic
permalink: seu-ecommerce-tao-rapido-quanto-o-google-usando-elasticsearch
#banner_image: sample-banner-image-1.jpg
meta_description: 'Seu e-commerce tão rápido quanto o Google usando ElasticSearch'
browser_title: 'Seu e-commerce tão rápido quanto o Google usando ElasticSearch'
---

Imagine a seguinte cena: uma loja virtual com um enorme catálogo de produtos. Após uma bem elaborada estratégia de vendas para atrair o público-alvo para o site, o consumidor finalmente chega até este e-commerce. E mais, ele está disposto a gastar. E quando tudo indicava um desfecho sensacional (conversão!!), o usuário não consegue encontrar na loja o produto que tanto precisa e desiste da compra.

E agora, o que devemos fazer? Uma solução é utilizar um mecanismo de FullTextSearch do MySQL que, aparentemente, irá resolver o problema por um tempo. Mas, digamos que o CEO de sua loja virtual resolva fazer uma mega liquidação, no Black Friday (por exemplo), e para turbinar os acessos e vender ainda mais, promove uma mega campanha no Google e Facebook. De repente, o fluxo de pessoas procurando desesperadamente por promoções no site aumenta 10x. É ai que começa a complicar para você que resolveu utilizar o FullTextSearch.

Será que você estava de fato preparado para enfrentar esta situação? Acho que você não vai querer surpresas e descobrir da pior maneira que não estava pronto para todo esse fluxo. Então, é justamente neste ponto que o ElasticSearch irá te ajudar.

### O ElasticSearch

O ElasticSearch é um mecanismo de busca, Open Source, construido em cima do Apache Lucene, poderoso motor de busca full-text. Este recurso conta com uma amigavél RESTFul API, dados em tempo real, alta disponibilidade, documentos de orientação, entre outras características.

Empresas como GitHub, Twitter, Google, Ebay, FourSquare, Bloomberg, The Guardian, Globo.com, Yelp e Moip já usam o ElasticSearch em produção para buscar e agregar em tempo real.

### Colocando o ElasticSearch para funcionar

A instalação deste mecanismo é bem simples. Basta fazer download da última versão<https://www.elastic.co/downloads>, descompactar e rodar bin/elasticsearch. Ou, acessar bin/elasticsearch.bat e em fazer uma requisição 

{% highlight bash linenos %}
$ curl -XGET http://localhost:9200/
{% endhighlight %}

### Alguns conceitos antes de começar

Para facilitar a comprensão de alguns conceitos e termos, elaboramos uma tabela comparativa do ElasticSerarch com um banco de relacional (MySQL):

| MySQL         | ElasticSearch |
| ------------- |-------------- |
| Database      | Index         |
| Table         | Type          |
| Row           | Document      |
| Column        | Field         |
| Schema        | Mapping       |
| Partition     | Shard         |

Outro conceito muito importante é entender como ele armazena os seus documentos. Para quem já utilizou um banco de dados NoSQL, como MongoDB, não irá ter problema em compreender. Este mecanismo utiliza a estrutura de JSON, suportada pela a maioria das linguagens de progração.

### Criando os primeiros registros

Para melhor acompanhamento dos seguintes passos, você pode utilizar um cliente REST de sua preferência. Neste exemplo, vamos utilizar um curl para simplicidade.

Modelo para inserir dados no ElasticSearch:

{% highlight bash linenos %}
$ curl -X PUT http://localhost:9200/produtos/biclicletas/1 -d '{
    "modelo": "speed",
    "nome": "Specialized Tarmac",
    "marchas": 14,
    "cor": "azul",
    "tags": [
        "bike",
        "speed",
        "specialized",
        "tarmac",
        "14 marchas"
    ],
    "valor": 1500000
}'
{% endhighlight %}

O que fizemos foi o seguinte: no Index produtos no Type bicicletas adicionamos a uma bicicleta, onde o id dela é 1. Caso você não forneça um Id para o documento do ElasticSearch, este irá gerar um id para você.

Você terá a seguinte resposta do ElasticSearch:

{% highlight json linenos %}
{
    "_index": "produtos",
    "_type": "bicicletas",
    "_id": "1",
    "_version": 1,
    "created": true,
    "_source": {
        "modelo": "speed",
        "nome": "Specialized Tarmac",
        "marchas": 14,
        "cor": "azul",
        "tags": [
            "bike",
            "speed",
            "specialized",
            "tarmac",
            "14 marchas"
        ],
        "valor": 1500000
    }
}
{% endhighlight %}

Se você desejar buscar o registro acima, basta executar a seguinte requisição:

{% highlight bash linenos %}
$ curl -XGET http://localhost:9200/produtos/biclicletas/1
{% endhighlight %}

Existem duas maneiras de realizar bucas no ElasticSearch. A mais simples é apenas utilizando query strings na URL da requisição. Utilizamos esta forma geralmente para pesquisas mais simples e rápidas.

A outra maneira para queries mais complexas é enviando um JSON com a Query DSL. Em geral utilizamos este quando queremos resultados mais refinados ou fazer agregações entre outras features do Elasticsearch.

Agora, vamos supor que você queira buscar por bicicletas azuis, utilizando Query String:

{% highlight bash linenos %}
$ curl -XGET http://localhost:9200/produtos/biclicletas/_search/?q=cor:azul
{% endhighlight %}

Na query acima enviamos o Field cor com a cor desejada. No caso azul, assim seguinte query irá procurar por todas as bicicletas que tenha a cor azul. O ElasticSearch irá retorna o seguinte resultado:

{% highlight json linenos %}
{
    "took": 3,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 0.30685282,
        "hits": [
            {
                "_index": "produtos",
                "_type": "bicicletas",
                "_id": "1",
                "_score": 0.30685282,
                "_source": {
                    "modelo": "speed",
                    "nome": "Specialized Tarmac",
                    "marchas": 14,
                    "cor": "azul",
                    "tags": [
                        "bike",
                        "speed",
                        "specialized",
                        "tarmac",
                        "14 marchas"
                    ],
                    "valor": 1500000
                }
            }
        ]
    }
}
{% endhighlight %}

Para realizar a mesma busca utlizando Query DSL

{% highlight bash linenos %}
$ curl -X GET http://localhost:9200/produtos/bicicletas/ -d '{
    "query": {
        "match": {
            "cor": "azul"
        }
    }
}'
{% endhighlight %}

### Conclusão

Neste artigo abordamos apenas uma introdução ao ElasticSearch. Ainda incluem features como geolocation, sugestão, analytics, entre outras. Outro ponto interessante é que o ElasticSearch possui diversas libs para facilitar sua utilização em sua linguagem de progração favorita, vale a pena visitar o site deles e confirir.
