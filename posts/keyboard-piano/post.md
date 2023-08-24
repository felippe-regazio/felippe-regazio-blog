<!--:::{
  "post_title": "Transformando seu teclado num pianinho infantil com algumas linhas de JS",
  "post_description": "Apenas uma brincadeira simples com vanilla js e oscillators",
  "post_created_at": "Thu Aug 24 2023 00:17:24 GMT-0300 (Horário Padrão de Brasília)"
}:::-->

O código abaixo transforma seu teclado em um Piano. Vá no ALT da imagem, copie o código, cole no seu console, clique num lugar vazio da pagina, e só apertar as teclinhas rs. Se travar em um som, aperte "n" pra mutar. As teclas sonoras serão `A...L` e `Q...P`.

```javascript
const getKeyFrequency = n => Math.pow(2, (n - 49) / 12) * 440;
const audioCtx = new AudioContext();
const playing = {};

window.addEventListener('keydown', e => {
  'qawsedrftgyhujikolp'.split('').forEach((k, i) => {
    if (e.key === k && !playing[k]) {
      const frequency = getKeyFrequency(30 + i);
      const oscillator = audioCtx.createOscillator();
      playing[k] = oscillator;
      oscillator?.frequency.setValueAtTime(frequency, audioCtx.currentTime);
      oscillator.connect(audioCtx.destination);
      oscillator?.frequency.setValueAtTime(frequency, audioCtx.currentTime);
      oscillator.type = "triangle";
      oscillator.start();
    }
  });
});

window.addEventListener('keyup', e => {
  const k = e.key;
  
  const stop = x => {
    try {
      playing[x].stop();
      playing[x].disconnect;    
      playing[x] = false;
    }catch{}
  }
  
  if (e.key === 'n') {
    Object.keys(playing).forEach(x => stop(x));
  }
  
  stop(k);
});
```

getKeyFrequency define a frequencia de cada tecla com n = 30+x. faça a conta e vc tera a frequencia da primeira tecla, agora só ver a que nota ela pertence. adicione +1 na equação pra cada prox. tecla e vc saberá as notas. mude o 30 e vc muda a escala, assim que "afina" isso rs.

Se preferir, clique no botão abaixo:

<button onclick="piano()">Piano</button>

<script>
  const getKeyFrequency = n => Math.pow(2, (n - 49) / 12) * 440;
  const audioCtx = new AudioContext();
  const playing = {};

  window.addEventListener('keydown', e => {
    'qawsedrftgyhujikolp'.split('').forEach((k, i) => {
      if (e.key === k && !playing[k]) {
        const frequency = getKeyFrequency(30 + i);
        const oscillator = audioCtx.createOscillator();
        playing[k] = oscillator;
        oscillator?.frequency.setValueAtTime(frequency, audioCtx.currentTime);
        oscillator.connect(audioCtx.destination);
        oscillator?.frequency.setValueAtTime(frequency, audioCtx.currentTime);
        oscillator.type = "triangle";
        oscillator.start();
      }
    });
  });

  window.addEventListener('keyup', e => {
    const k = e.key;
    
    const stop = x => {
      try {
        playing[x].stop();
        playing[x].disconnect;    
        playing[x] = false;
      }catch{}
    }
    
    if (e.key === 'n') {
      Object.keys(playing).forEach(x => stop(x));
    }
    
    stop(k);
  });
</script>