<!--:::{
  "post_title": "Enviando eventos para diferentes abas de um mesmo domínio sem precisar de um Backend",
  "post_description": "Imagine que vc tem diversas abas de um mesmo site abertas e vc quer enviar um evento pra todas essas abas sem precisar de nenhum backend, pure front",
  "post_created_at": "Wed Nov 09 2022 15:28:26 GMT-0300 (Brasilia Standard Time)"
}:::-->


Imagine que vc tem diversas abas de um mesmo site abertas e vc quer enviar um evento pra todas essas abas sem precisar de nenhum backend, pure front. 

Ex: user clicou em "Sair" e todas as abas abertas devem mostrar uma mensagem discreta: "Usuário deslogado"

Existem algumas formas de fazer, mas não tão simples quanto a que vou mostrar aqui:

Vamos utilizar o evento "storage" da window.

Esse evento escuta toda vez que um dado é modificado na storage (local/session) é modificado num mesmo contexto: https://developer.mozilla.org/en-US/docs/Web/API/Window/storage_event

O que significa isso: Quer dizer que se eu tenho 10 abas de um site abertas e eu modificar a local storage em qualquer uma dessa abas, o evento "storage" será imediatamente executado nas 10 abas.

Isso nos permite utilizar o evento "storage" pra criar um sistema de "broadcast" que funciona inteiro via Front End, sem a necessidade de Backend. 

Embora bem limitado, pode ser bem util pra todo tipo de notificação efemera, como o usuário "sair" e todas as abas saberem disso.

Basicamente o evento "storage" é descrito dessa forma:

```js
window.addEventListener("storage", (e) => { ... });
```

Toda vez que a storage é modificada vamos escutar e receber algumas informações como qual chave foi modificada, o valor antigo e o valor novo da nossa modificação.

Isso é o suficiente pra construirmos uma função "listener". É essa função que vai escutar as modificações da nossa storage. Ou seja: essa função vai escutar nossas "mensagens".

```js
const BROADCAST_TRANSFER_KEY = 'broadcast-message';

function listen () {
  if (!window.BROADCAST_LISTENER) {
    window.BROADCAST_LISTENER = window.addEventListener('storage', (e) => {
      if (e.key !== BROADCAST_TRANSFER_KEY ) {
        return false;
      }
        
      const message = JSON.parse(e.newValue);
      
      if (message && message.broadcast) {
        alert(message.payload);
      }
    });
  }
}
```

Pra acessar o código acima basta clicar no "ALT" contido na imagem e vc vai conseguir copia-lo como texto. 

Essa função basicamente escuta nossa storage, e toda vez que a chave "broadcast-message" for modificada no nosso contexto ela executará nosso callback.

O callback por sua vez faz o parse do dos dados modificados na chave "broadcast-message" da storage, e verifica se os dados inseridos tem um parametro broadcast: true (por segurança). Se sim, dá um alert passando os dados de "payload".

Pra testar é simples:

1. Abra varias abas de um mesmo site (umas 3).  

2. Abra o console, cole esse codigo em todas e chame a função listen() em seguida pra ativar nosso listener

Agora vamos enviar mensagens.

Agora que vc executou o listener em todas as suas abas, para enviar uma mensagem vc vai precisar dessa função (Clique no parametro ALT pra pega-la em modo texto).

Obs: A BROADCAST_TRANSFER_KEY foi declarada no snippet anterior. Se precisar, declare ela de novo com o mesmo valor

```js
function send (payload) {
  localStorage.setItem(BROADCAST_TRANSFER_KEY, JSON.stringify({ 
    payload, 
    broadcast: true 
  }));

  localStorage.removeItem(BROADCAST_TRANSFER_KEY);
}
```

Agora va em qualquer uma das abas que vc deixou abertas (as mesmas em que colou o snippet com a função listen)

Cole essa nova função (send) e chame-a enviando uma mensagem, ex:

```js
send("Hello from the other side!");
```

Pronto, agora visite as outras abas e veja a mágica acontecer 😊 Agora fica fácil mandar uma mensagem de usuário deslogado pra todas as abas, por ex, né haha.

O browser possui uma API específica para esse fim via Worker também, caso você queira saber mais: https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API