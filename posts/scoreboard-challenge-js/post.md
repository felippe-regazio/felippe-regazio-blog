<!--:::{
  "post_title": "Resolvendo e explicando um desafio intermediário de JavaScript, passo a passo",
  "post_description": "Resolução passo a passo de um desafio JavaScript intermediário e fornecimento de contexto a respeito da história e evolução do ecossistema JS",
  "post_created_at": "Fri Oct 21 2022 11:56:40 GMT-0300 (Horário Padrão de Brasília)"
}:::-->

## O Desafio

Imagine que estou apostando corrida de carrinhos Hot Wheels com meu filho, e preciso de uma função JS pra registrar o placar. 

O problema é que eu só tenho o Internet Explorer de 2010 pra fazer essa função, então tenho algumas regras

- não posso utilizar let nem const
- não posso manipular o escopo externo
- não posso utilizar nenhum mock
- devo utilizar vanilla js pra construir
- devo utilizar UMA função global
- não posso utilizar frameworks
- não posso utilizar transpilers

O comportamento esperado é:

```js
var score = scoreboard({ felippe: 0, lorenzo: 0 });

score();
```

output:

```
Felippe: 0
Lorenzo: 0
```

Para marcar pontos precisaremos de um metodo

```js
score.point('felippe');

score();
```

output:

```
Felippe: 1
Lorenzo: 0
```

E eu posso marcar mais de um ponto se preciso, basta usar um segundo parametro para incrementar mais de um ponto

```js
score.point('lorenzo', 3);

score();
```

output:

```
Felippe: 1
Lorenzo: 3
```

E a qualquer momento da brincadeira nós podemos adicionar novas pessoas:

```js
score.add('flavia');

score();
```

output:

```
Felippe: 1
Lorenzo: 3
Flavia: 0
```

E também podemos adicionar essa pessoa com uma pontuação inicial se preciso:

```js
score.add('Joana', 2);

score();
```

output:

```
Felippe: 1
Lorenzo: 3
Flavia: 0
Joana: 2
```

### Mais regras

Eu posso inicializar o score com mais de 2 pessoas mas nunca somente com uma

Eu não posso somar pontos para uma pessoa que não existe no placar

Você pode ter uma única função global, mas pode ter quantas funções quiser dentro dela

### Bonus (opcional):

As manipulações que requerem que nomes sejam passados devem ser case INSENSITIVE, e o output deve ser escrito em lowercase com a primeira letra do nome maiúscula.

ex:

score.point('FeLiPPe'); // ponto pra felippe

### Dicas

https://stackoverflow.com/questions/20705063/javascript-object-keys-method-alternative-for-compatibility-with-ie8

https://stackoverflow.com/questions/690251/what-happened-to-console-log-in-ie8

Boa sorte!

## Resolução

Primeiramente temos que considerar que faremos um código vanilla JS para IE10. Sendo assim muitas das funcionalidades modernas e syntax sugar do JS não estarão mais a mão. A primeira coisa que perdemos ao pensar logo de cara no código: `classes`. Mas peraí, `classes` em JS são em grande parte syntax sugar para funções. Então podemos começar criando nossa main function:

```js
function scoreboard(initialPeople) {
  //
}
```

De acordo com a especificação do desafio, a função `scoreboard` recebe um objeto com a estrutura `{ [key: string]: number }` e retorna uma nova função que controla este objeto. Portanto `scoreboard` é uma `HOF: Higher Order Function`. Funções que manipulam outras funções (seja recebendo como parametro ou retornando funções) são chamadas de `HOF`. Em JavaScript quando você cria uma dessas, o escopo da função mãe é preservado mesmo após retornamos outra função, e é dessa tecnica que vamos nos beneficiar. Isso significa o seguinte:

```js
function scoreboard(initialPeople) {
  var value = 0;

  return function(n) {
    value += (n || 0);
    console.log("Value is now: "+value);
  }
}
```

No exemplo acima nós declaramos um valor no escopo de `scoreboard` e retornamos uma função que manipula esse valor. Se executarmos esse código, teremos o seguinte comportamento:

```js
var score = scoreboard({});

// como scoreboard retorna uma função, score é uma function
score();
// Value is now: 0
score(3);
// Value is now: 3
score(2);
// Value is now: 5
score(10);
// Value is now: 15
```

Como observado, scoreboard é uma HOF que retorna uma função. A função retornada permanece ligada ao escopo da função mãe. Assim podemos manipular `value` de forma cumulativa, pois o escopo é preservado. Isso é justamente a ideia central da nossa `scoreboard`. 

Pensando nisso, vamos então fazer um dos primeiros requisitos do nosso desafio: a função que adiciona uma pessoa nova ao jogo. Basicamente significa que vamos adicionar uma nova chave com um novo valor ao que foi passado em `initialPeople`. Como sabemos o funcionamento de uma HOF, isso agora fica fácil. Os passos são:

- Verífica se pessoa já existe
- Se existe: gera warn e retorna false
- Se não existe adiciona

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  return function(person, points) {
    if (people[person]) {
      console.log("A person named '" + person + "' is already playing.");

      return false;
    }

    people[person] = (points || 0);
  }
}
```

Está feita nossa função. Veja que assinamos os parametros em `initialPeople` para uma nova variável `people`. Fazemos isso porque vamos modificar os valores originais, e é uma boa prática não modificar diretamente os parametros da função.

Depois disso verificamos se existe uma chave com o nome passado em nosso objeto, se sim, avisamos e não fazemos nada. Se não, adicionamos a chave com um novo valor ao objeto. Se nenhum ponto foi passado, a pessoa começa com 0. Se testarmos:


```js
var score = scoreboard({});

score('Felippe', 1);
``` 

Bom, de cara você já deve ter visto o problema: a notação da nossa função está errada. Se voltarmos aos requisitos do desafio, ele espera que para adicionar uma nova pessoa façamos o seguinte:

```js
score.add('flavia');
score.add('Joana', 2);
```

Para fazermos isso precisamos saber de outra questão conceitual: funções são objetos em javascript. Um objeto pode conter uma chave e um valor, logo uma função também. Quando aplicarmos uma chave e um valor a uma função como se ela fosse um objeto, chamamos esse par `chave: valor` de propriedade. Por exemplo, se quisermos adicionar uma propriedade a nossa função score precisamos antes:

- criar nossa função com um nome
- assinar novas propriedades a função

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  function score(person, points) {
    return "fizz";
  }

  score.customProp = 1;

  return score;
}
```

Acima então criamos a função `score` que retorna "fizz", e adicionamos a ela a propriedade `customProp` cuja chave é homonima (customProp) e o valor é '1'. Veja:

```js
var score = scoreboard({});

score(); // -> "fizz"
console.log(score.customProp); // 1
```

Podemos assinar diferentes tipos de valores a uma propriedade, inclusive uma função. Quando criamos uma propriedade em uma função e o valor dessa propriedade será outra função, nós temos um `método`:

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  function score(person, points) {
    return "fizz";
  }

  score.myMethod = function() {
    return "buzz"
  };

  return score;
}
```

Acima nossa função score retorna `fizz`, e possui o método `myMethod` que retorna `buzz`:

```js
var score = scoreboard({});

score(); // -> "fizz"
score.myMethod(); // -> "buzz"
```

Sabendo disso, agora podemos arrumar nossa função `scoreboard` para ter o método `add` que adiciona pessoas ao objeto inicial:

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  function score() {
    //
  }

  score.add = function(person, points) {
    if (people[person]) {
      console.log("A person named '" + person + "' is already playing.");

      return false;
    }

    people[person] = (points || 0);    
  }

  return score;
}
```

Pronto, agora temos um método add. Mas ainda temos dois problemas: verificações diretas como `people[person]` poderiam ser inseguras em determinadas versões do IE. Como não sabemos se o IE sofre disso mas somos precavidos, vamos criar um método que verifica se uma chave existe em dado objeto e usar ele pra fazer essa verificação em `add`. Como antigamente também não tínhamos a global `Object` para manipularmos objetos, vamos ter que usar o método `hasOwnProperty` diretamente. O método `hasOwnProperty` verifica justamente se determinada propriedade existe em um determinado objeto.

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  function keyExists(key, obj) {
    return hasOwnProperty.call(obj, key);
  }

  function score() {
    //
  }

  score.add = function(person, points) {
    if (keyExists(person, people)) {
      console.log("A person named '" + person + "' is already playing.");

      return false;
    }

    people[person] = (points || 0);    
  }

  return score;
}
```

E temos um segundo problema: Algumas versões do IE só permitiam que usássemos `console.log` quando o inspector estivesse aberto no browser, do contrário a função não existiria no escopo global gerando um erro no código. Pra mitigar esse problema podemos criar uma função `log` nossa que faz um `try`. Ou seja: ela tenta usar `console.log`, se der tudo bem (é porque o inspector estava aberto), mas se não der tudo bem também hahaha (IE Rules):

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  function log(s) {
    try { console.log(s) } catch {};
  }

  function keyExists(key, obj) {
    return hasOwnProperty.call(obj, key);
  }

  function score() {
    //
  }

  score.add = function(person, points) {
    if (keyExists(person, people)) {
      log("A person named '" + person + "' is already playing.");

      return false;
    }

    people[person] = (points || 0);    
  }

  return score;
}
```

Pronto, ufa! Agora parece estar tudo certo. Temos nosso método `add` e podemos testá-lo:

```js
var score = scoreboard({});

score(); // void
score.add("felippe", 2); // void
```

Mas como vamos saber se está tudo funcionando sem ver como está nosso objeto `{people}`? Poderíamos usar a nossa função `log` pra inspecionar, mas seria melhor se tivéssemos mesmo uma função que mostra o placar, né? Problema é que quando chamamos a função `score()`, ela que deveria mostrar o placar ainda não faz nada. Então vamos fazer a função que mostra o placar:

## Mostrando o placar

Para mostrar o placar nós precisaremos:

- de um objeto válido
- passar item por item desse objeto
- printar a chave + valor do objeto

Ou seja, precisaremos de iteração. Iteração é a arte de passar item por item de um dado conjunto. No nosso caso vamos passar item por item do nosso objeto. Se estivessemos numa versão mais recente de JavaScript poderíamos utilizar `Object.keys`, `Object.values`, `Object.entries`, etc. para iterar propriedades de um objeto. Porém não estamos, então teremos que escrever nosso próprio iterador.

Para criar nosso iterador utilizaremos o loop `for in`, que é compatível com IE:

```js
function scoreboard(initialPeople) {
  /* (...) */

  function objEntries(obj, callback) {
    for (key in obj) {
      if (keyExists(key, obj)) {
        callback(key, obj[key]);
      }
    }
  }  
  /* (...) */

  return score;
}
```

Observe que no código acima criamos a função `objEntries`. Esta função passa item por item de um objeto, assegura que a chave realmente existe e chama um callback passando `chave, valor` como argumentos. Ou seja: é basicamente um `forEach`. Vamos então utiliza-la para printar nosso objeto como solicitado na especificação do desafio:

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  function log(s) {
    try { console.log(s) } catch {};
  }

  function keyExists(key, obj) {
    return hasOwnProperty.call(obj, key);
  }

  function objEntries(obj, callback) {
    for (key in obj) {
      if (keyExists(key, obj)) {
        callback(key, obj[key]);
      }
    }
  }  

  function score() {
    objEntries(people, function(key, value) {
      log(key + ": " + value);
    });
  }

  score.add = function(person, points) {
    if (keyExists(person, people)) {
      log("A person named '" + person + "' is already playing.");

      return false;
    }

    people[person] = (points || 0);    
  }

  return score;
}
```

O que dissemos na função score então é o seguinte: para cada `key: value` no objeto `people` execute um `console.log`. A saída seria:

```js
var score = scoreboard({ felippe: 0 });

score.add("lorenzo", 2);

score();

// felippe: 0
// lorenzo: 2
```

Ótimo, agora podemos ver nossas funções em ação. Mas se observarmos, os nomes estão no padrão que foram escritos na chave do objeto. O ideal seria se normalizássemos para o padrão: Primeira letra maiúscula, o resto em minúsculo.

## Normalizando Strings

Vamos agora fazer uma função que converte uma string para o padrão "Primeira letra maiúscula, o resto em minúscula`, ou seja: Capitaliza uma string.

Pode não parecer mas nesse caso temos duas tarefas, e seria ideal que usassemos funções diferentes para cada uma delas.

1. Deixar uma string toda em minúsculo
2. Deixar apenas a primeira letra maiúscula

Então mãos a obra. Faremos primeiro a função que converte uma string para lowercase:

```js
function lc(str) {
  if (!str) return "";
  
  return str.toLowerCase();
}
```

Simples assim: veríficamos se a string existe, se não retorna string vazia, do contrário chama `toLowerCase`. Basicamente criamos uma sintax sugar. Agora vamos usar isso para criar uma função `ucFirst`, que capitalizará uma string pra nós:

```js
function ucFirst(str) {
  if (!str) return "";

  return str.charAt(0).toUpperCase() + lc(str.slice(1));
}
```

Pronto. Utilizamos `charAt` para compatibilidade com IE. Pegamos o caractere de index zero (primeiro caractere) e passamos pra uppercase, e utilizamos nossa lc para noramlizar o resto pra lower case.

Agora vamos acoplar nossas funções na nossa lib, e usar elas pra normalizar o output do método `score`:

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  function log(s) {
    try { console.log(s) } catch {};
  }

  function keyExists(key, obj) {
    return hasOwnProperty.call(obj, key);
  }

  function objEntries(obj, callback) {
    for (key in obj) {
      if (keyExists(key, obj)) {
        callback(key, obj[key]);
      }
    }
  }

  function lc(str) {
    if (!str) return "";
    
    return str.toLowerCase();
  }

  function ucFirst(str) {
    if (!str) return "";

    return str.charAt(0).toUpperCase() + lc(str.slice(1));
  }

  function score() {
    objEntries(people, function(key, value) {
      log(ucFirst(key) + ": " + value);
    });
  }

  score.add = function(person, points) {
    if (keyExists(person, people)) {
      log("A person named '" + ucFirst(person) + "' is already playing.");

      return false;
    }

    people[person] = (points || 0);    
  }

  return score;
}
```

Observe acima que adicionamos as duas funções de manipulação de strings ao nosso modulo, e depois chamamos a função `ucFirst` na saída do método `score`. Agora temos:

```js
var score = scoreboard({ felippe: 0 });

score.add("lorenzo", 2);

score();

// Felippe: 0
// Lorenzo: 2
```

Não importa o quão fora de padrão um nome seja adicionado a nossa coleção de pessoas, a saída dele será sempre normalizada.

## Organização importa

Se você olhar para o código que escrevemos até aqui, perceberá que temos funções de duas naturezas diferentes nele:

1. Funções internas, que não são facilmente acessíveis
2. Funções externas, facilmente acessíveis

A primeira opção damos o nome de `métodos privados`. Alguém utilizando nossa lib não conseguiria acesso fácil as funções internas delas, como por exemplo as recém criadas `lc` e `ucFirst`. Estas funções dizem respeito ao maquinário interno do nosso módulo e não tem porque alguém acessá-las.

Por outro lado temos a função `score` que é retornada, e que possui varios métodos próprios. Esta por sua vez é uma função publica. Também é comum chamar esse tipo de retorno de `API Pública do Módulo`. Dizemos isso porque é através dessa função que o módulo oferece funcionalidades e são essas funções que estarão a mão para o consumer.

Para facilitar a compreensão e melhorar a organização do nosso código, vamos adicionar um comentário que demarcará uma fronteira entre `privado x público`

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  function log(s) {
    try { console.log(s) } catch {};
  }

  function keyExists(key, obj) {
    return hasOwnProperty.call(obj, key);
  }

  function objEntries(obj, callback) {
    for (key in obj) {
      if (keyExists(key, obj)) {
        callback(key, obj[key]);
      }
    }
  }

  function lc(str) {
    if (!str) return "";
    
    return str.toLowerCase();
  }

  function ucFirst(str) {
    if (!str) return "";

    return str.charAt(0).toUpperCase() + lc(str.slice(1));
  }

  // ------ Public ------

  function score() {
    objEntries(people, function(key, value) {
      log(ucFirst(key) + ": " + value);
    });
  }

  score.add = function(person, points) {
    if (keyExists(person, people)) {
      log("A person named '" + ucFirst(person) + "' is already playing.");

      return false;
    }

    people[person] = (points || 0);    
  }

  return score;
}
```

## Incrementando a pontuação

Agora vamos fazer uma função que adiciona pontos para uma pessoa. Se observarmos, essa função é uma variação simples do nosso método de add, porém os concerns dela são:

1. Verifica se a pessoa existe
2. Se sim, soma os pontos passados aos pontos da pessoa

Vamos adicionar logo após a função `score.add` uma nova função: `score.point`. Logo temos:

```js
score.point = function(person, points) {
  if (!keyExists(person, people)) {
    log("There is no person named '" + ucFirst(person) + "' in the game.");

    return false;
  }

  var newValue = (people[person] || 0) + (points || 1);
  people[person] = newValue;
}
```

Veja que a execução é bem simples: Se a pessoa recebendo pontos não existir, avisa e não faz nada. Se a pessoa existir pega os pontos atuais dela e soma aos novos pontos passados no argumento `points`. Se não foi passado um valor para points, soma 1 ponto. Pega os novos pontos e assina como sendo o novo valor para a pessoa no objeto `people`.

Para testar:

```js
var score = scoreboard({ felippe: 0 });

score.add("lorenzo", 2);
score();

// Felippe: 0
// Lorenzo: 2

score.point('felippe', 1);
score();

// Felippe: 0
// Lorenzo: 2

score.point('joana', 3);
// There is no person named 'Joana' in the game.
```

## Quase lá

A esta altura já concluímos quase todos os requisitos do desafio. Então temos duas pendencias de bonus para concluir:

1. Nomes passados como parametro devem ser case-insensitive
2. Não podemos inicializar um `scoreboard` com zero ou uma pessoa

### Aplicando Case-Insenstive Inputs

A ideia aqui é que as operações com nomes sejam case insensitive para que evitemos nomes duplicados escritos de forma diferente, por ex:

```
Felippe
FeLIPPE
FELIPPE
feliPpe
``` 

Para a nossa lib, todos os nomes acima devem ser o mesmo: `felippe`. Para isso temos que seguir um princípio: Todo input e comparação entre strings devem ser normalizadas. Você pode normalizar como quiser desde que sempre utilize a mesma normalização. No nosso caso vamos usar `lowercase`, e temos 3 pontos mapeados onde devemos normalizar nossas strings de forma a deixar a lib case-insenstive:

1. Parametro `initialPeople`
2. Método público `point`
3. Método público `add`


Ou seja: não importa como o usuário passa um nome para nós, este nome sempre vai ser salvo e manipulado em lower-case, dessa forma garantimos que tudo estará case-insenstive.

Para nossa felicidade nós já escrevemos um método que converte uma String pra lowercase, o `lc`. Então podemos utilizá-lo no item 1 (método point) e 2 (método add) da nossa lista. Lembre-se de não manipular o parametro original da função. 

> Warn: nem sempre é uma boa ideia mutar objetos passados como parametro mesmo que não seja diretamente. Isso porque existe uma ideia cognitiva associada a eles. Esse exercício teve um caráter mais pedagógico (e também teve algum sentido par ao que estamos fazendo). Mas não tome esse tipo de prática como regra, entenda que esse tipo de coisa deve ser analisada com cautela e raramente é necessária.

Até aqui nossa lib está assim:

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  function log(s) {
    try { console.log(s) } catch {};
  }

  function keyExists(key, obj) {
    return hasOwnProperty.call(obj, key);
  }

  function objEntries(obj, callback) {
    for (key in obj) {
      if (keyExists(key, obj)) {
        callback(key, obj[key]);
      }
    }
  }

  function lc(str) {
    if (!str) return "";
    
    return str.toLowerCase();
  }

  function ucFirst(str) {
    if (!str) return "";

    return str.charAt(0).toUpperCase() + lc(str.slice(1));
  }

  // ------ Public ------

  function score() {
    objEntries(people, function(key, value) {
      log(ucFirst(key) + ": " + value);
    });
  }

  score.add = function(person, points) {
    if (keyExists(person, people)) {
      log("A person named '" + ucFirst(person) + "' is already playing.");

      return false;
    }

    people[person] = (points || 0);    
  }

  score.point = function(person, points) {
    if (!keyExists(person, people)) {
      log("There is no person named '" + ucFirst(person) + "' in the game.");

      return false;
    }

    var newValue = (people[person] || 0) + (points || 1);
    people[person] = newValue;
  }

  return score;
}
```

Vamos refatorar o método `add` para:

```js
score.add = function(person, points) {
  var lcPerson = lc(person);

  if (keyExists(lcPerson, people)) {
    log("A person named '" + ucFirst(person) + "' is already playing.");
    
    return false;
  }

  people[lcPerson] = (points || 0);
}
```

E o método `point` para:

```js
score.point = function(person, points) {
  var lcPerson = lc(person);

  if (!keyExists(lcPerson, people)) {
    log("There is no person named '" + ucFirst(person) + "' in the game.");

    return false;
  }

  var newValue = (people[lcPerson] || 0) + (points || 1);
  people[lcPerson] = newValue;
}
```

Bom, como vc pôde ver, todos os nossos dados de input estão normalizados, com exceção do objeto `initialPeople`. Isso porque teremos que tratar ele de forma especial: vamos iterar esse objeto gerando um novo objeto normalizado.

## Normalizando o parametro `initialPeople`

Até aqui, nosso objeto people (o qual é manipulado por nossa lib) é inicialmente derivado diretamente do dado que o consumer passou pra nós:

```js
function scoreboard(initialPeople) {
  var people = initialPeople;

  /** ... **/
}
```

Neste ponto, o que queremos que aconteça é: a variavel people deve conter os dados de inicialPeople, porém com todas as keys (nomes de pessoas) normalizadas. Pra isso vamos criar uma função que itera o objeto e converte as keys para lower case, retornando um novo objeto:

```js
function objKeysToLower(obj) {
  var result = {};

  objEntries(obj, function(key, value) {
    result[lc(key)] = value;
  });

  return result;
}
```

Pronto, agora basta utilizar nosso novo método para normalizar o valor em `people`, que deriva de `initialPeople`:

```js
function scoreboard(initialPeople) {
  var people = objKeysToLower(initialPeople);

  /** ... **/
}
```

Pronto, agora não importa como o consumer insira os dados, nossa lib vai sempre lidar com eles em formato lowercase.

## Validando o objeto `initialPeople`

Agora nos resta uma ultima pendencia: validar existe 2 ou mais pessoas passadas inicialmente ao iicializar nossa lib. Para isso vamos fazer nosso último método que conta o número de chaves em um Objeto. Note que esta função é uma variação da função `objKeysToLower`, mas invés de gerar um novo objeto, nós geramos um array com as keys do objeto atual ou um array vazio caso o parameto não seja um objeto;

```js
function objKeys(obj) {
  // Este if utiliza uma tecnica antiga para 
  // assegurar que um valor é um objeto e não é um Array
  if (typeof obj !== 'object' || hasOwnProperty.call(obj, 'length')) {
    return [];
  }

  var result = [];

  objEntries(obj, function(key) {
    result.push([lc(key)]);
  });

  return result;
}
```

Pronto, agora antes de inicializar nossa lib, nós validamos se temos um `initialPeople` valido:

```js
function scoreboard(initialPeople) {
  if (objKeys(initialPeople).length <= 1) {
    throw Error('You need at least two people to start');
  }

  var people = objKeysToLower(initialPeople);

  /** ... **/
}
```

Se tivermos um parametro inicial inválido, lançamos um erro.

## Resultado

Pronto, nossa lib está feita e contempla todos os requisitos passados pelo desafio:

```js
function scoreboard(initialPeople) {
  if (objKeys(initialPeople).length <= 1) {
    throw Error('You need at least two people to start');
  }

  var people = objKeysToLower(initialPeople);

  function log(s) {
    try { console.log(s) } catch {};
  }

  function keyExists(key, obj) {
    return hasOwnProperty.call(obj, key);
  }

  function objEntries(obj, callback) {
    for (key in obj) {
      if (keyExists(key, obj)) {
        callback(key, obj[key]);
      }
    }
  }

  function lc(str) {
    if (!str) return "";
    
    return str.toLowerCase();
  }

  function ucFirst(str) {
    if (!str) return "";

    return str.charAt(0).toUpperCase() + lc(str.slice(1));
  }

  function objKeysToLower(obj) {
    var result = {};

    objEntries(obj, function(key, value) {
      result[lc(key)] = value;
    });

    return result;
  }

  function objKeys(obj) {
    if (typeof obj !== 'object' || hasOwnProperty.call(obj, 'length')) {
      return [];
    }

    var result = [];

    objEntries(obj, function(key) {
      result.push([lc(key)]);
    });

    return result;
  }  

  // ------ Public ------

  function score() {
    objEntries(people, function(key, value) {
      log(ucFirst(key) + ": " + value);
    });
  }

  score.add = function(person, points) {
    var lcPerson = lc(person);

    if (keyExists(lcPerson, people)) {
      log("A person named '" + ucFirst(person) + "' is already playing.");
      
      return false;
    }

    people[lcPerson] = (points || 0);
  }

  score.point = function(person, points) {
    var lcPerson = lc(person);

    if (!keyExists(lcPerson, people)) {
      log("There is no person named '" + ucFirst(person) + "' in the game.");

      return false;
    }

    var newValue = (people[lcPerson] || 0) + (points || 1);
    people[lcPerson] = newValue;
  }

  return score;
}
```

Você pode rodar os casos abaixo para testar:


```js
var score = scoreboard({ felippe: 0, lorenzo: 0 });

score.point('felippe');
score.point('lorenzo', 2);
score.add('flavia');
score.add('joana', 3);
score.point('fLaViA', 3);

score();
// Felippe: 1
// Lorenzo: 2
// Flavia: 3
// Joana: 3
```

Você pode rodar os casos baixo para testar verificações de error:

```js
var score = scoreboard({});
// VM394:3 Uncaught Error: You need at least two people to start
// at scoreboard (<anonymous>:3:11)
// at <anonymous>:1:15

var score = scoreboard({ felippe: 0 });
// VM441:3 Uncaught Error: You need at least two people to start
// at scoreboard (<anonymous>:3:11)
// at <anonymous>:1:15

var score = scoreboard({ felippe: 0, lorenzo: 0 });
score.point('joana', 10)
// VM472:9 There is no person named 'Joana' in the game.
// false
```

Embora bastante funcional, nosso código ainda possui infinitas imperfeições que você pode querer ou não arrumar. Porém o importante aqui é que ele cumpriu um papel principal: Ser didático. Como é um código de complexidade média, existe ainda margem para muita refatoração. Sinta-se livre pra brincar com isso.

## Conclusão

A ideia aqui nunca foi escrever um código pra IE na verdade, essa era apenas uma desculpa para examinarmos conceitos e práticas de base. Não importa se o código roda no IE ou não porque o IE morreu, o que importa é que escrevendo esse código com essa premissa você foi forçado/a a verificar como funciona cada aspecto do que você estava fazendo. Logo você aprendeu muitas coisas novas sobre o que você já sabia.

Ninguém precisa saber escrever código assim hoje em dia, e mesmo se souber, ainda utilizaríamos os recursos mais recentes ainda assim. Porém, as tecnicas e conceitos demonstrados aqui podem ser utilizados em qualquer código, em qualquer aplicação. Por exemplo: depois desse exercício você não olhará para uma classe da mesma forma nunca mais, você saberá a dinamica entre Objetos privados e públicos dessa classe, propriedades e a relação delas com seu escopo. Você olhará para componentes React sabendo que são Higher Order Functions dentro de outras Higher Order Functions, cada um podendo manter e manipular escopos internos com propriedades (estados) próprias, etc.

Outro ponto importante sobre este exercício: Atualmente os diversos frameworks e syntax sugars escondem complexidades em métodos mágicos, isso nos dá velocidade e simplifica semanticamente o código, porém no momento em que um bug ocorre os motivos podem ficar ocultos e obscuros dentro desses mesmos métodos mágicos. Portanto, conhecer conceitos de base e o maquinário que derivou o que utilizamos hoje é importante para debugarmos e entendermos melhor as relações entre os componentes que utilizamos para contruir nossas aplicações.

Uma sugestão de estudo a partir daqui seria: reescreva essa mesma lib em ES6 utilizando classes. Você verá na prática como o ecossistema JavaScript evoluiu, e o mais importante: o motivo disso. As diferenças de escopo e comportamento entre var, let e const. As diferenças de escopo entre functions e arrow functions. Os comportamentos gerados pelo hoisting, etc.

<br/><br/><br/>