<!--:::{
  "post_title": "Como funciona o teclado dinâmico de senhas do Banco?",
  "post_description": "Acho que essa curiosidade ou duvida já deve ter passado na cabeça de alguns/mas devs por aí. Vou tentar responder nessa thread de maneira bem resumida uma das técnicas de fazer um teclado desse.",
  "post_created_at": "Mon Nov 06 2023 15:33:44 GMT-0300 (Horário Padrão de Brasília)"
}:::-->

<image src="./teclado.jpeg" alt="Imagem da UI de um teclado para input de senha em aplicativo de banco" />

O ponto central desse problema é ser capaz de gerar todas as combinações numericas de um array de arrays numericos. Se você pensar assim, depois de resolver isso, você vai gerar um novo array com todas as combinações numéricas possiveis geradas pelos botões (arrays) escolhidos.

Este é um problema de "permutação", você vai ter que permutar todos os arrays até gerar todas as combinações, depois ver se elas contem a senha esperada. Você pode usar a seguinte função para permutar:

```js
function cartesian(...args) {
  var r = [], max = args.length - 1;

  function helper(arr, i) {
    for (var j=0, l=args[i].length; j<l; j++) {
      var a = arr.slice(0);
      a.push(args[i][j]);
      i === max ? r.push(a) : helper(a, i+1);
    }
  }

  helper([], 0);
  return r;
}
```

O front fica responsavel apenas por coletar os botões apertados e enviar ao backend. Voê pode enviar como array, range, ID de ranges, whatever... Vamos supor que sua senha seja "1234". Para acertar essa senha você apertaria (e o front enviaria):  `[1, 2]; [1, 2]; [3, 9]; [4, 6]`.

No backend nós gerariamos as permutações com nossa função:

```js
const passwords = cartesian([1, 2], [1, 2], [3, 9], [4, 6]);
```

A saída para a chamada acima seria:

```js
[
  [ 1, 1, 3, 4 ],
  [ 1, 1, 3, 6 ],
  [ 1, 1, 9, 4 ],
  [ 1, 1, 9, 6 ],
  [ 1, 2, 3, 4 ],
  [ 1, 2, 3, 6 ],
  [ 1, 2, 9, 4 ],
  [ 1, 2, 9, 6 ],
  [ 2, 1, 3, 4 ],
  [ 2, 1, 3, 6 ],
  [ 2, 1, 9, 4 ],
  [ 2, 1, 9, 6 ],
  [ 2, 2, 3, 4 ],
  [ 2, 2, 3, 6 ],
  [ 2, 2, 9, 4 ],
  [ 2, 2, 9, 6 ]
]
```

E depois verificaríamos se a nossa senha existe em algum dos indexes do array gerado dinamicamente. No nosso caso seria o index (4). Essa é a lógica básica. Levando em consideração um hash de senha salvo na database, teríamos um fluxo completo.

## Fluxo ilustrativo

A título de ilustração, faríamos algo assim:

```js
// keylist: [[1, 2], [1, 2], [3, 9], [4, 6]];
const passargs = request.body.keylist;

// passamos os arrays apertados pelo usuario como parametro
const possiblePasswords = cartesian(...passargs);

// função imaginaria que pega o hash de senha do usuario
const userPassHash = await DB.getUserPasswordHash(userId);

// chacamos se algum item da lista de senha contem o hash do usuario
for (item of possiblePasswords) {
  // transform items no padrão [0, 0, 0, 0] em "0000"
  const pass = item.join('');
  // cria um hash com o item atual da lista de possiveis senhas
  const hash = hashPass(pass);
  // compara o hash do item com o do user: userPassHash
  if (hash === userPassHash) {
    // usuario esta logado, finaliza o loop e nem chaca o resto
    send200('Success');
    break;
  } else {
    // se não deu, tenta comparar o próximo item
    continue;
  }
}

// se tentou todos e não houve sucesso, fim de fluxo
send401('Not allowed');
``` 

Acima temos um código imaginário que ilustra de forma bem simplista o fluxo de login levando o hash em conta, usamos uma estrutura análoga a de uma API REST para facilitar a compreensão, porém esse fluxo pode aparecer em SPA, MPA, ou qualquer outro lugar. O hash de senha dependerá da escolha de cada projeto.

## Security Concerns

O texto acima ilustra o fluxo de maneira simplista para compreensão da tecnica utilizada. Porém podemos otimiza-lo do ponto de vista de segurança em diversos aspectos. Alguns dos mais comuns são:

1. Toda vez que uma tentativa de login é iniciada, o backend inicia uma sessão opaca, e gera um keyboard para essa sessão contendo o mapa do teclado que será apresentado para o user. O backend envia esse mapa para o front.

2. O front interpreta o mapa, e para cada array de keys será atribuído um hash dinâmico. Ex:

[1, 2] // 8743b52063cd84097a65d1633f5c74f5

Assim, quando o user apertar [1, 2] o front marcará como sendo "8743b52063cd84097a65d1633f5c74f5" e uma coleção de hashes será enviada no final para o backend, que terá o mapa de keys para essa sessão e fará o processo reverso, verificando que 8743b52063cd84097a65d1633f5c74f5, por exemplo, equivale a [1, 2] naquela sessão.

Isso invalidade ataques de brute force. Também evita ataques man-in-the-middle, uma vez que um interceptador com o payload de login descriptografado em mãos e sem um snapshot do front, teria apenas uma coleção de hashes. Como os mapas mudam para cada sessão (para cada tentativa de login), mesmo que esse atacante reaproveite esses hashes e os envie para logar, os hashes terão sido invalidados, tenha o login obtido sucesso ou não.

Esse é apenas um dos exemplos de otimizações de segurança aplicadas em um fluxo como esse. Elas podem se extender por muitas camadas, inclusive de hardware ou até biológica (validação biométrica junto a senha, por exemplo).

## Hashing vs Encrypting

Quando recebemos uma senha de usuário nós não salvamos ela no banco de dados diretamente, nós criamos um hash dessa senha. Um hash consiste em, a partir de um dado, criar uma versão cifrada desse mesmo dado que nunca mais possa ser decifrada (ou ao menos não deveria). Ou seja, para uma senha 1234 criariamos um hash e salvariamos esse hash no banco de dados, algo assim "01dfae6e5d4d90d9892622325959afbe".

Fazemo o processo acima para proteger uma informação crítica do usuário a qual não dependemos e nem precisamos saber. Assim, um sistema que siga esse fluxo não sabe sua senha. Ele sabe apenas o hash gerado a partir dela. É comum ainda utilizarmos um "salt", ou seja, um outro hash que juntamos a sua senha para gerar um terceiro hash e salvar no banco de dados. Assim, caso seu banco de dados vazasse com a senha de todos os usuarios, elas não seriam expostas, e mesmo que um atacante tentasse força bruta para desfazer o hash e ver a senha, ele não conseguiria pois não teria o salt para intuir o padrão.

Mas se não sabemos a senha do usuário, como logamos ele? Bom, pedimos uma "prova de conhecimento". Isso funciona assim: O usuário manda a senha para o backend, ele digita "1234", nós criamos novamente um hash e verificamos se o hash criado no momento do login é igual ao que criamos para o usuario no dia em que ele escolheu a senha dele. Se a senha digitada formar um hash igual ao que está no banco de dados, o usuário digitou a senha correta.

Mas isso não é criptografia? Mais ou menos. Encriptar um dado consiste no processo de cifrar esse dado e obter uma chave que possa fazer o caminho reverso: transformar a cifra no dado original novamente, ou seja: ler a mensagem. Esse não é o caso de um hash, um hash não deve ser desfeito.

## Conclusão

O intuito desse post foi explicar a tecnica por trás do TECLADO e validação de vetores de senha no contexto bancário. Ele não explica o fluxo de login inteiro, completo e produtivo, mas o core disso, e a partir daí você já sabe como funciona o password-matching e pode evoluir a tecnica conforme suas próprias necessidades.