<!--:::{
  "post_title": "Mockando APIs diretamente no Front End utilizando apenas funções simples",
  "post_description": "'Oh não meu mundo caiu, não posso continuar minha task pq tenho os contratos mas a API não tá pronta'. Deixa de ser chorão Frontender. Neste post vou mostrar uma das tecnicas pra mockar comportamento de requests diretamente no front.",
  "post_created_at": "Thu Aug 24 2023 00:06:18 GMT-0300 (Horário Padrão de Brasília)"
}:::-->

Existem diversos serviços, dependencias, packages e afins que fazem mock de APIs desde as formas mais complexas as mais simples. Porém, se vc quer algo super rapido a mão, aqui vão algumas dicas com Vanilla JS:

Essa é nossa factory, ela "finge" ser uma request, e retorna os dados que definirmos em data, seta as opções na response, e o tempo de resposta como sleep. Vc pode modificar de acordo com suas necessidades.

```javascript
function mockRequest(data, options, sleep = 200) {
    const response = new Response(JSON.stringify(data), options);
    
    return new Promise((resolve, reject) => {        
        setTimeout(() => {
            options.status >= 400 ? reject(response) : resolve(response);
        }, sleep)
    });
}
```

Agora vc pode usar seu fetch mocado pra fingir comportamentos e retornos de requests no front, desafogando sua task. Quando o backend terminar, vc só substitui por um rest-client real com chamadas pra uma api real. Ex de request mocada com nossa factory:

```javascript
mockRequest({ fizz: 1 }, { status: 200 })
  .then(resp => resp.json())
  .then(console.log);

// Promise {<pending>}
// { fizz: 1 } 
```

Ou seja, no codigo acima dizemos que vamos mockar uma requisição que vai retornar um json { fizz: 1 } com status 200, e na sequencia recebemos o pseudo-comportamento  assíncrono de request. Assim seria um error:

```javascript
mockRequest({ buzz: 1 }, { status: 400 })
  .catch(console.error);

// Promise<{pending}>
// <Console error info>
```

Lembrando que isso é apenas a demonstração da tecnica, vc pode modificar a factory pra agir da forma que vc quiser e retornar os mocks como vc quiser. Podemos até fingir timeouts. Aqui uma """request""" que demora 8 secs ora resoonder e volta um error:

```javascript
mockRequest(null, { status: 408 }, 8000)
  .catch(console.error);

// Promise<{pending}>
// Wait 8 secs
// <Console error info>  
```

Logo, se vc ver que uma API vai demorar pra ficar pronta, o que vc precisa é pegar com o backend os contratos: Ex - O que ela retorna como sucesso? Como os erros são retornados. Agora crie um modulo de mocks no seu projeto e use a tecnica acima pra mockar as requests.

O interessante dos mocks tb, quando em trampos mais complexos, é que vc pode forçar muitos cenarios como:

- Erros da familia 40X e 50X
- Timeouts e latencias
- Body quebrada ou erronea
- Ver se os contratos fazem sentido
- Pegar bugs rapidamente no front

O parametro options na nossa factory aceita qualquer opção do construtor de Response: https://developer.mozilla.org/en-US/docs/Web/API/Response/Response

Vc pode inclusive evoluir essa tecnica pra receber parametros como se fosse de "request" e modificar a Response de acordo com eles, e até mesmo modificar qual metodo a response vai fingir ser, que headers que vai setar, cors, etc. Leia sobre o construtor de Response.

Obs: avisar nunca é demais: primeiro veja os requisitos e os contratos, só então escreva a sua propria factory. Ex: error code é respeitado ou é só 200 com error na body? mock isso tb. Mas no final a tecnica é essa: Uma promise, um sleep e um construtor de Response já dá conta.

Agora, se vc vai mockar apenas uma GET Request e quer muito usar fetch de vdd, dá tbm. Sim, direto no front. Vamos criar uma Object URL Dinamica que retorna seu JSON:

```javascript
function createURLThatReturns(data) {
  const json = JSON.stringify(data);
  const blob = new Blob([json], {type: "application/json"});
  
  return URL.createObjectURL(blob);
}
```

Pronto, agora vc pode criar URLs que retornam os dados JSON que vc definir. Criando uma URL que recebe um GET e retorna { fizz: 1 }

```javascript
const url = createURLThatReturns({ fizz: 1 });
```

E usar essa URL pra fazer um GET de vdd e receber { fizz: 1 }.

```javascript
const url = createURLThatReturns({ fizz: 1 });

fetch(url)
  .then(resp => resp.json())
  .then(console.log);

// Promise<{ pending }>
// { fizz: 1 }
```

Vc pode mockar quantos GETs quiser, mas lembre-se de que a URL é dinamica, ela morre junto com a runtime, então utilize referencias (consts) como no ex acima. É isso.