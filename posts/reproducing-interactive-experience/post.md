<!--:::{
  "post_title": "Video Série: Criando uma aplicação interativa do zero com HTML/CSS/JS puros",
  "post_description": "Nessa série em vídeo pretendo explicar todo o processo de construção de uma tela interativa, focando em padrões de mercado, conceitos de programação e e arquitetura",
  "post_created_at": "Fri Aug 09 2024 20:16:36 GMT-0300 (Brasilia Standard Time)"
}:::-->

<style>
  video {
    max-width: 100%;
    margin: 32px auto;
    display: block !important;
  }
</style>

O Designer Musab Alfawal (@musabalfawal) postou no Twitter (X) um vídeo demonstrando uma tela (UI/UX) que ele prototipou (Provavelmente no Figma). A tela abaixo:

<video controls>
  <source src="video01.mp4" type="video/mp4">
</video>

E por diversão decidi recriar esse comportamento da maneira mais aproximada possível utilizando tecnologias Vanilla somente (basicamente um arquivo HTML com CSS e JS puros, sem lib, sem framework).

É claro que sem acesso ao protótipo de design muitas coisas se perderam como algumas proporções, espaçamentos, font family, icons, etc. Por se tratar de um port de uma tela mobile para o browser, alguns aspectos da experiência também foram reduzidos como animações pregressivas acompanhando o slider. Porém o resultado foi bastante satisfatório pra uma brincadeira feita "de olho". Veja:

<video controls style="display: block; margin: 32px auto">
  <source src="video02.mov" type="video/mp4">
</video>

Ao mostrar esse vídeo algumas pessoas se interessaram no processo de coding dessa tela, por isso decidi fazer essa série em vídeo explicando todos os aspectos de criação dessa experiência utilizando apenas tecnologias Vanilla:

## 00. Source Code and Live Version

Você pode acessar todo o código fonte da aplicação nesse repositório: <a href="https://github.com/felippe-regazio/notbad" target="_blank">https://github.com/felippe-regazio/notbad.</a>

Para uma versão live dessa experiência acesse <a href="https://felippe-regazio.github.io/notbad/" target="_blank">https://felippe-regazio.github.io/notbad/.</a>

## 01. Fundamentos, Conceitos e Arquitetura Inicial

Para acessar a video-aula 01 sobre a construção dessa tela, acesse o link:
https://drive.google.com/file/d/1HsnGQx7TddNVgTUjTbjO1n5PsQrTQRZe/view?usp=sharing

## 02. Comportamentos iniciais

Under Construction
