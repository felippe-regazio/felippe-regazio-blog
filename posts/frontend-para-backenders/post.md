
<!--:::{
  "post_title": "Frontend para Backenders | Conceitos necessários",
  "post_description": "Ao longo da minha carreira eu percebi que muitos backs não sabem como o front moderno funciona, e isso gera vários problemas de arquitetura e várias tretas de escopo nos apps.",
  "post_created_at": "Thu Feb 23 2023 12:20:54 GMT-0300 (Brasilia Standard Time)"
}:::-->

Ao longo da minha carreira eu percebi que muitos backs não sabem como o front moderno funciona, e isso gera vários problemas de arquitetura e várias tretas de escopo nos apps.

Então segue o que a Back PRECISA saber sobre o Front moderno:

## CORS

Essa é sem dúvida uma dor. Então basicamente o que é o CORS? Vamos a historia:

Antigamente qualquer client poderia requisitar qualquer recurso de qualquer domain...

Se eu criasse um site e colacasse nele imagens do seu site usando suas URLs, tudo certo. Vc meio que ia pagar servidor pra mim, eu ia consumir seus recursos. Barrar isso era complicado. Então o CORS foi criado pra evitar isso.

CORS quer dizer: Cross-Origin Resource Sharing (CORS) - Compartilhamento de Recurso de Domínio-Cruzado.

Basicamente ele delimita QUEM pode pedir o que para determinado server. Essa camada é ativada em requests de um browser-client para um server.

Falando assim parece que isso é problema do front né? Então: não é. Assim que o front mandar uma request para o server, isso vai acontecer:

O server vai ler a "Origin" e vai retornar uma header determinando quem pode ler aquele recurso, a: Access-Control-Allow-Origin.

Se o front não estiver permitido ali a request é barrada pelo BROWSER. Não há nada que um frontender possa fazer a não ser pedir pro backender configurar corretamente as headers de CORS. E não adianta testar no Postman pq de novo: é o BROWSER que vai barrar.

Por padrão o CORS vem configurado como same-origin. Isso quer dizer que um dominio só pode pedir recurso para o mesmo domínio. Ou seja: site x carrega imagem de site x, se carregar de site Y: CORS Block!

Problema é que hoje usamos APIs desacopladas que rodam em outros domínios. Quando o front X fizer request para a API Y, se a API Y não configurou a Access-Control-Allow-Origin para esse Front, o browser vai barrar a request com CORS-Block. Cabe ao BACKEND configurar o CORS da API

## SPA/PWA

Antigamente toda vez que vc acessava uma pagina, uma request era feita para um servidor que retornava o conteudo dela para o browser desenhar o HTML e prover os recursos. Hoje isso não funciona mais assim na maior parte das vezes.

SPA significa: Single Page Application. Isso significa que quando vc abrir um front o que será carregado vai ser um Bundle JS todo fatiado. Aquilo é basicamente a aplicação inteira. É quase como "instalar" um app no navegador. A partir daí tudo é carregado DINAMICAMENTE

Quando sua pagina for carregada no front, é o front geralmente que vai:

- controlar as rotas (URLs e como elas mudam, o backe nem vai enxergar)

-  quase toda request para API será dinamica via http-request e cross domain

- se vc mudar de pagina o FRONT carrega ela

Na pratica isso quer dizer que não adianta o back fazer malabarismos de segurança, gateways, sessões mágicas que definem estado de login baseado em requests, redirects rebuscados pq: NAO VAI FUNCIONAR rs.

Cabe ainda citar que: http-requests são stateless, o front só vai conhecer seu backend no momento da request, assim que a request terminar, seu backend não sabe que o client existe nem vice-versa.

Se vc decidir INSTALAR mesmo essa aplicação no browser, vc tem uma PWA: Progressive Web App. Ou aplicação web progressiva, isso pq ela carrega toda a casca e vai pedindo recursos pro server PROGRESSIVAMENTE. Ou seja: o back tem pouca governança sobre o funcionamento do Front.

## 🟨 BFF

Com esse monte de regras pode ser que o front comece a se meter muito no backend e até a forçar certos modelos. Isso pode ser ruim, daí que entra o "Back For Front", uma camada backend criada exclusivamente pra fazer a ponte entre um Client e uma API complexa.

Em organizações mais complexas, com times de backends complexos e grandes um BFF salva muito o tempo dos fronts e dá MUITA liberdade pra ambos os lados, bem como deixa o proprio front cuidar de aspectos de back inerentes ao Front. pra saber mais:

<a href="https://felipperegazio.com/posts/vamos-falar-de-bff-pattern/" target="_blank">Vamos falar de BFF Pattern?</a>

## 🟨 URL/DOMINOS IMPORTAM!

Muitas vezes em requests de microservices para microservices, um domino ou URL é um mero endpoint para um recurso, e não dita nada mais que isso. (as vezes não, mas geralmente URL pro back são endpoints, não são contratos).

Para o front a URL é um contrato. Ela é atrelada a uma cacetada de políticas e verificações:

CORS, CSP, Cookie Policies, Roteamento de SPAs, . Numa SPA então o Front terá pleno domínio da URL, historico e roteamento via Client mesmo.

Leve isso em conta, dependendo de onde um back coloca um recurso, se muda ele de lugar, ou se resolve enxer de parametros, queries, hashes, isso pode influenciar profundamente o front. Nada ali é "só mover do dominio X pro Y que quando o front bater vai dar no mesmo". Não vai rs.

## 🟨 Dado vem, dado volta (same model)

Não importa como Forms são construídos no front, uma coisa é certa: Da mesma forma que eles vieram eles tem que voltar pra garantir uma boa UX e uma boa DX.

Isso deve-se a varios motivos estruturais que não vou discorrer aqui, mas uma coisa deve ser feita:

Se um recurso é editavel, o back deve procurar fechar um contrato com o front e não fugir dessa estrutura. Ex: Vc vai fazer uma todo list com os seguintes endpoints

GET /items { <items> }

PUT /item { itemID, itemDescription }

Vc tem um endpoint que lista e um que insere um novo item. Logo vc teria o de editar, certo?

Para esses endpoints a estrutura de item deve ser sempre a MESMA. Todos os items retornados em /items devem ser:

 { itemID, itemDescription }

O dado que vai via PUT?

 { itemID, itemDescription }

O que vai pra editar?

 { itemID, itemDescription }

 O contrato pode ter campos decrementais (opcionais), mas não deve mudar de estrutura. Se o contrato muda de acordo com a ação nos endpoints para um mesmo recurso o front vai ter que fazer uma cacetada de malabarismo pra entender quando o user ta tomando cada ação (criar, editar, etc) e mudar o modelo de acordo com isso: bugs a vista e complexidade desnecessária.

## Thats all folks

Importante ter em mente que o front virou um monstrinho, não é mais aquela coisa desarticulada e simplista. Pense numa aplicação Front com basicamente a mesma (ou mais) complexidade que uma aplicação desktop  rodando num SO chamado Browser.
