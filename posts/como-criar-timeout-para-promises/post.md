<!--:::{
  "post_title": "Como adicionar Timeout em Promises não confiáveis",
  "post_description": "Algumas promises podem nunca resolver, a depender de como são construídas ou do que estão fazendo, se possuem unreacheable code, callbacks perdidos, etc etc. Nesse caso vc teria um memory leak e problemas de fluxo... Vc vai ter que arrumar anyway, mas colocar um timeout ajuda",
  "post_created_at": "Tue Feb 28 2023 16:11:52 GMT-0300 (Brasilia Standard Time)"
}:::-->

Algumas promises podem nunca resolver, a depender de como são construídas ou do que estão fazendo, se possuem unreacheable code, callbacks perdidos, etc etc. Nesse caso vc teria um memory leak e problemas de fluxo... Vc vai ter que arrumar anyway, mas colocar um timeout ajuda.

Isso significa que a Promise terá o tempo que vc estipular para ser resolvida ou rejeitada, se nada disso acontecer, ela será automaticamente rejeitada. Pra fazer isso usamos "Promise.race"

Eis o código:

```javascript
/**
 * Adiciona um callback (tieout) para qualquer promise
 * Função "timeout" que recebe qualquer promise como parametro, e um "time" em ms
 * 
 * A função cria uma chamada Promise.race passando nossa "promise" e em seguida
 * passando uma Promise dinamica que é automaticamente rejeitada após o "time"
 * estipulado; como a Promise.race resolve para a promise que for processada
 * primeiro, se nossa promise demorar mais que o tempo estipulado o callback
 * dinamico vai triggar um reject resolvendo o racing e liberando a meória
 * 
 * Exemplos de uso:
 * 
 * timeout(new Promise(() => {}), 1000);
 * timeout(new Promise((r) => r(true)), 1000);
 * const anything = await timeout(new Promise(() => {}), 1000);
 * 
 * @param promise : Promise<any>
 * @param time : Number
 * @returns Promise<unknown>
 */
const timeout = (promise, time) => Promise.race([
  promise,
  new Promise((_, reject) => setTimeout(() => reject('Promise timedout'), time))
]);
```

Promise.race aceita um array de Promises e resolve o valor da Promise que se resolver primeiro. Pra saber mais sobre Promise.race: <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race" target="_blank">MDN Promise.race()</a>

No nosso snippet nós passamos para o Promise.race a nossa promise (callback) e criamos uma segunda Promise dinamica que será rejeitada após o tempo estipulado. Ou seja: nós forçamos nossa primeira Promise a resolver no timeout especificado ou vamos trigar um reject a força haha

O que acontece é que se nossa promise original não resolver ou falhar, a que setamos com timeout vai trigar um reject e o race vai ser resolvido, liberarando a memória.

Uso:

```javascript
timeout(new Promise((resolve, reject) => { ... }), 5000);
```

A promise acima terá 5 seg pra ser processada (seja com sucesso ou não) ou será liberada da fila de execução pelo timeout trigado via Promise.race.

Vc pode usar essa tecnica com qualquer tipo de "Promisable", inclusive awaits.

```javascript
const anything = await timeout(new Promise((resolve, reject) => { ... }), 5000);
```

É isso.