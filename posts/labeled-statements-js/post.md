<!--:::{
  "post_title": "Labeled Statements em JS",
  "post_description": "Labeled Statements são escopos etiquetados que vc pode abrir dentro de qualquer bloco em JS.",
  "post_created_at": "Thu May 04 2023 16:34:23 GMT-0300 (Brasilia Standard Time)"
}:::-->

Sem pesquisar, sem olhar no console, sem checar nada, simplesmente batendo o olho na função JS abaixo, o que ela vai retornar?


```javascript
function fn() {
  const value = 0;
  
  inner: {
    const value = 5;
  }

  return value;
}
```

Bom, primeiro vamos falar sobre algo que observamos pouco: escopos. Em JS a notação { ... } é usada para definir o inicio e fim de um Statement.

No caso de uma função:

```javascript
function fn() { return 1 };
```

Estamos dizendo que temos uma função chamada "fn" que não tem argumentos, e em seguida iniciamos a function statement, que terá seu próprio escopo, dizemos que ela retornará 1 e finalizamos o statement.

Você consegue ler um pouco sobre isso <a href="https://www.w3schools.com/jsref/jsref_function.asp" target="_blank">Aqui</a>.

O que vc viu acima é a Function Statement, ou simplesmente "declaração de funcão". Às vezes o statement é reduzido e dependendo da notação vc não precisa definir o início e fim, ele simplesmente será inferido. Ex:

```javascript
const fn = () => 1;
```

O que temos acima é uma function expression, que é muito similar a declaração de função, porém aceita expressões auto-retornáveis, iifes, anonymous functions, etc. Por isso age como expressão

Você pode saber mais sobre isso <a href="https://developer.mozilla.org/en-US/docs/web/JavaScript/Reference/Operators/function" target="_blank">Aqui</a>.

A essas horas vc deve estar se perguntando: "Ok mas o que isso tem a ver com aquele esquisitissimo inner: {} lá no meio da função, e que pior ainda, não retornou nenhum erro mesmo duplicando uma declaração de const?

Bom, acontece que vc pode delimitar um Statement na hora que vc quiser em JS desde que vc esteja dentro de um escopo. E na vdd ele nem precisa de um nome. O delimitador { ... } não precisa de nada mais pra abrir um statement além de estar dentro de um escopo. Confuso?

Vamos lá, veja o exemplo abaixo:

```javascript
function fn() {
  const value = 0;

  console.log(value);

  {
    const value = 5;

    console.log(value);
  }

  return value;
}
```

O que está dentro dos {  ...  } será interpretado como um novo statement, dentro de um novo escopo

No caso acima, se vc rodar em algum lugar, terá a saida

```javascript
0 // primeiro console log, escopo da fn
5 // segundo console log, escopo do inner statement
0 // valor do return value
```

A const não conflita porque nosso novo statement define um novo escopo. Porém veja o que acontece agora.

Observe a variação abaixo:

```javascript
function fn() {
  const value = 0;

  console.log(value);

  {
    console.log(value);
  }

  return value;
}
```

O retorno será acima será

```javascript
0; // console log, escopo fn
0; // console log, escopo inner statement
0; // return
```

O que isso quer dizer? Que escopos podem ser aninhados, e eles tem acesso a dados do parent scope, embora possam declarar os seus próprios sem conflitar.

Porém, escopos externos não tem acesso a valores dos escopos internos, veja:

```javascript
function fn() {
  console.log(typeof value);

  {
    const value = 0;

    console.log(typeof value);
  }

  console.log(typeof value);
}
```

Utilizamos typeof pra evitar erro de acesso a valor não declarado. Aí que tá, se vc executar o valor acima, terá o output.

```javascript
undefined // console log, escopo fn
number // console log escopo inner statement
undefined // console log, escopo fn
```

O escopo externo não acessa o interno.

Tá, mas até agora nada do "inner:" né? O que diabos é isso?

Nada mais é que um label para o escopo. Daí o nome "Labeled Statement". Vc pode nomear um escopo ao inicia-lo:

```javascript
function fn() {
  const value = 0;
  
  inner: {
    const value = 5;
  }

  return value;
}
```

A função acima tem um "labeled statement" chamado inner. Agora tudo fez sentido né? Juntando com tudo o que foi explicado até aqui, aposto que agora vc sabe exatamente o que está acontecendo.

<a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/label" target="_blank">Doc para saber mais sobre labeled statements.</a>

Labeled Statements aceitam ações parecidas com um "GOTO" em determinados contextos, e por isso, dentre outras coisas, é que existem.

A existencia dos statements é consequencia do funcionamento e da engine do próprio JS, não é necessariamente uma feature.

Entretando, se vc tentar abrir um anonymous statement, ou um labeled statement fora de um scopo valido (dentro de uma expressão por exemplo), umas merdas feias vão acontecer. A notação {} dentro de uma expressão será interpretada como um Objeto por ex.

> WARN: ESTE É UM POST INFORMATIVO: Eu desencorajo fortemente e declaradamente o uso de `anonymous statements` e `labeled statements`. Eles são efeitos da construção da lang, ou features antigas (no caso de labels). Não consigo ver motivo genuído pra utilização.