<!--:::{
  "post_title": "Backend para Frontenders, o essencial",
  "post_description": "Esses são alguns aspectos essenciais de backend que não vão tornar vc Backer, mas vão tornar vc um/a frontender melhor",
  "post_created_at": "Wed Apr 26 2023 00:07:25 GMT-0300 (Horário Padrão de Brasília)"
}:::-->


Este post visa listar conteúdos comumente pertinentes ao backend mas que agregam valor para frontenders. Esse valor pode ser tecnico ou não, como por exemplo: melhorar comunicabilidade, agilidade, tolerancia, entrosamento, etc. 

Este post é voltado para Plenos/Seniores.

## Desenho Arquitetural

Se vc tem um escopo arquitetural ou pegou um "greenfield" aí pra começar uma aplicação que envolve backend, faça alguns desenhos arquiteturais pra demonstrar os contratos e fluxo de comunicações. Backenders se orientam muito assim (e estão corretos).


Muitas vezes o front fica orientado a layout e figma e acaba negligenciando desenhos arquiteturais, fluxogramas para fluxo de dados e a construção das aplicações, bem como o fluxo de comunicação dela com o back. Não é sempre que precisa, mas quando precisa não é sempre que tem rs

## Databases (SQL/NoSQL)

Aprenda o básico de databases, sugiro:

MySQL/MariaDB/PostGres/Redis/Dynamo/Mongo

Quando digo o básico é: acesso remoto, queries básicas de pesquisa e filtro de dados, um mínimo de teoria relacional, alguns usecases pra cada uma. Ajuda muito!

## Aprenda o básico de alguma/s langs de back

Sugiro Node/JS, GoLang, mas tbm podem ser PHP / Java / Ruby etc. 

Sugiro que vc tente construir alguma aplicaçãozinha básica, comumente uma API REST. Sua visão vai expandir bastante no front ao fazer isso.

## Comunicação Client Server

Os assuntos chave aqui são: Básico de como funcionam requests http, status codes comuns e seus significados, ready state e suas fases, CORS, stateless vs statefull e cache.  Ou seja, procure esmiuçar como seu front conversa com o Back.

## Básico de estruturas de Rede/Infra

Entenda o básico da infra em que sua aplicação está rodando, tanto front quanto back, se tem ou não gateways, se é um monolito ou microserviços, em que ponto da sua infra o front está e o quão próximo ou não ele está do backend

## Autenticação

Estude OAuth, JWT, leia sobre SAML, SSO e processos de autenticação em geral. Esse tipo de integração costuma ser uma dor tanto pro back quanto pro front. Procure entender isso com um olho no backend, vai ajudar bastante na conversa.

## Logs

Aqui é mais uma questão de hábito do que saber as coisas. Como front vc tbm pode e deve ler logs e ter acesso a eles, inclusive os logs do back (pra debugar front ajuda muito). Vc não precisa começar a debugar aspectos de back pro back, mas ler/acessar logs é preciso!

## CLI

Aprenda a usar a CLI. Nem que for para não utiliza-la sempre. Vc vai trombar com muito backender dinossauro que só usa CLI, saber o mínimo vai te ajudar a não ficar desconfortável ou perder foco em um pair, vai te ajudar a se virar em terminais, consoles remotos etc.

## Postman / Curl / Trace

Aprenda a utilizar Postman e ferramentas relacionadas, alguma coisa de Curl / wget / traceroute (não sei como é no windows mas unix based esses são bem presentes). Isso ajuda muito na troca de info, as vezes o back manda um acesso ou snippet do postman

## É isso

Bom, esses são os pontos que considero chave e que vão fazer de vc um/a frontender melhor, e com certeza muito mais amigável aos backenders. 

Só o que citei acima já é coisa pra cacete, então não precisa sair vendo tudo de uma vez. Pegue um ponto e vá no seu tempo. Bons estudos!
