<!--:::{
  "post_title": "Joguinho de Digitar em 50 linhas de JS puro",
  "post_description": "Um joguinho divertido que roda em qualquer lugar :)",
  "post_created_at": "Wed Aug 23 2023 23:47:36 GMT-0300 (Horário Padrão de Brasília)"
}:::-->

Copie o código abaixo e cole no console de qualquer página para iniciar o joguinho. Clique na página e basta digitar as teclas conforme aparecem. Seu score está no title da tab. O "2" em startGame é a velocidade em segundos da mudança de teclas, varie esse numero para alterar a dificuldade. 

```javascript
function startGame(interval) {
    let char;
    let hits = 0;
    let errors = 0;
    
    const letters = 'abcdefghijklmnopqrstuvwxyz'.split('');
    const holder = document.createElement('div')
    
    Object.assign(holder.style, {
      position: 'fixed',  
      height: '100vh',
      width: '100vw',
      zIndex: 99999,
      top: 0,
      left: 0,
      fontSize: '200px',
      color: '#fff',
      display: 'grid',
      placeItems: 'center',
      backgroundColor: 'rgba(0,0,0,.5)'
    });

    document.body.append(holder);    
    
    const score = (h, x) => {
        holder.textContent = '';
        document.title = `hits: ${h} - errors: ${x}`;
    }
    
    window.addEventListener('keydown', e => {
       e.key.toLowerCase() === char
        ? score(++hits, errors) 
        : score(hits, ++errors);
    });

    setInterval(() => {
        char = letters[Math.floor(Math.random()*letters.length)];
        holder.textContent = char;
    }, interval * 1000);
}

startGame(2);
```

Se preferir, clique no botão abaixo pra iniciar o joguinho aqui mesmo:

<form onsubmit="start(); return false">
  <label>Velocidade em Secs</label>
  <input type="number" name="interval" value="2"/>
  <br/>

  <button type="submit">
    Iniciar
  </button>
</form>

<script>
  function start() {
    const interval = Number(document.querySelector('[name=interval]').value);

    function startGame(interval) {
        let char;
        let hits = 0;
        let errors = 0;
        
        const letters = 'abcdefghijklmnopqrstuvwxyz'.split('');
        const holder = document.createElement('div')
        
        Object.assign(holder.style, {
          position: 'fixed',  
          height: '100vh',
          width: '100vw',
          zIndex: 99999,
          top: 0,
          left: 0,
          fontSize: '200px',
          color: '#fff',
          display: 'grid',
          placeItems: 'center',
          backgroundColor: 'rgba(0,0,0,.5)'
        });

        document.body.append(holder);    
        
        const score = (h, x) => {
            holder.textContent = '';
            document.title = `hits: ${h} - errors: ${x}`;
        }
        
        window.addEventListener('keydown', e => {
          e.key.toLowerCase() === char
            ? score(++hits, errors) 
            : score(hits, ++errors);
        });

        setInterval(() => {
            char = letters[Math.floor(Math.random()*letters.length)];
            holder.textContent = char;
        }, interval * 1000);
    }

    if (typeof interval !== 'number' || interval < 0.5 || interval > 3) {
      alert('Escolha uma velocidade entre 0.5 e 3')
    } else {
      startGame(interval);
    }
  }
</script>