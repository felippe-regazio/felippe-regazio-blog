<!--:::{
  "post_title": "CSS-In-JS nativo com Object.assign e objetos simples",
  "post_description": "Vc pode assinar Objetos CSS (CSS in JS) nativamente em JS utilizando: Object.assign(element.style, <StyleObject>); e o motivo é a parte mais legal",
  "post_created_at": "Tue May 16 2023 13:38:30 GMT-0300 (Brasilia Standard Time)"
}:::-->

Pois é, vc pode assinar Objetos CSS (CSS in JS) nativamente em JS utilizando:

```javascript
Object.assign(element.style, <StyleObject>);
```

Ou seja, vc pode extender a propriedade style de qualquer elemento JS e essa propriedade será espelhada para o atributo style da tag. É melhor do que fazer ligação direta via `element.style = '...'` rs. Ex:

```javascript
// div sem estilo <div></div>
const div = document.createElement('div');

// estilizando a div
Object.assign(div.style, {
  display: 'flex',
  alignItems: 'center',
  justifyContent: 'center'
});

// pronto, atribuimos novos valores a interface 
// CSSStyleDeclaration que se refletirá em estilo na div:
console.log(div);
// <div style="display: flex; align-items: center; justify-content: center">
```

Porém se vc fizer atribuição direta de um objeto, não vai funcionar

```javascript
// criando uma div qualquer
// <div></div>
const div = document.createElement('div');

// Isso NÃO vai funcionar
div.style = {
  display: 'flex',
  alignItems: 'center',
  justifyContent: 'center'
};

// a saída seria <div></div>
console.log(div);
```

Isso tem uma explicação bacana: a propriedade "style" dos elementos é um objeto com uma interface `CSSStyleDeclaration`.

Usando o `Object.assign` vc extende o ObjectStyle de forma elegante e segura sem sobre-escrever a interface, ao invés de fazer override pra uma string com `element.style = '...'`.

Quando vc faz `element.style="display: flex"` vc faz o override da interface de style pra uma string que será diretamente refletida no estilo. E quando vc faz `element.style = {...}` vc faz o override da interface de style para um objeto simples que não será traduzido para o atributo style.

Vide bula: <a href="https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style" target="_blank">MDN: HTMLElement: style property</a>.