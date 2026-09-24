# Tabelas de Referência

Em muitas ocasiões nos deparamos com partes do código onde precisamos verificar o valor de uma variável, e tomar diferentes ações de acordo com esta variável. Exemplo:  

```javascript
function atacar(habilidade, alvo) {
  if (habilidade === "hadouken") {
    alvo.vida -= 30
  } else if (habilidade === "shoryuken") {
    alvo.vida -= 35
  } else if (habilidade === "treco treco furugen") {
    alvo.vida -= 45
  } else if (habilidade === "kamehameha") {
    alvo.vida -= 75
  } else if (habilidade === "masenko") {
    alvo.vida -= 90
  } else if (habilidade === "curar") {
    alvo.vida += 30
  } else if (habilidade === "furtar") {
    alvo.dinheiro -= 50
  }
}

atacar("treco treco furugen", mario)
```

No código acima, a linha `} else if (habilidade === valor) {` é repetida constantemente, bem como a linha `alvo.ação = novoValor` trazendo com isso alguns problemas:  

1. Se quisermos mudar a lógica de comparação em um item, temos de mudar em todos eles.
2. Todos os testes de condições são executados, independente da posição deles no código, ou se algum já foi validado antes, custando em performance.
3. A leitura e compreensão fica exaustiva e prejudicada.

Para códigos assim, podemos utilizar a estrutura de "Tabela de Referências", onde ao invés de utilizarmos a variável (`habilidade` no exemplo) como algo a ser comparado, ela passa a ser utilizada como um "endereço" que leva a execução do código direto para o que desejamos rodar.

```javascript
function reduzirVida(alvo, valor) { alvo.vida -= valor}
function reduzirDinheiro(alvo, valor) { alvo.dinheiro -= valor }

const habilidades = {
  ["hadouken"]: function(alvo) { reduzirVida(alvo, 30) },
  ["shoryuken"]: function(alvo) { reduzirVida(alvo, 30) },
  ["treco treco furugen"]: function(alvo) { reduzirVida(alvo, 30) },
  ["kamehameha"]: function(alvo) { reduzirVida(alvo, 30) },
  ["masenko"]: function(alvo) { reduzirVida(alvo, 30) },
  ["curar"]: function(alvo) { reduzirVida(alvo, -30) },
  ["furtar"]: function(alvo) { reduzirDinheiro(alvo, 50) }
}

function atacar(habilidade, alvo) {
  habilidades[habilidade](alvo)
}

atacar("curar", kirby)
```

No código acima, a variável `habilidade` é usada como um "endereço" para as propriedades do objeto `habilidades`. Notem que para isso funcionar temos algumas diferenças:

1. Normalmente em objetos, os nomes das propriedades seguem as regras para nomeação de variáveis do JavaScript (sem acentos, sem caracteres especiais, começa sempre com letra). Mas para uma tabela de referências isso não funciona por conta da forma como acessamos objetos:
    - `objeto.propriedade` = Estou acessando uma propriedade chamada "propriedade" neste objeto
    - `objeto[propriedade]` = Estou acessando uma propriedade cujo valor está armazenado em uma variável chamada `propriedade`
2. De forma similar, podemos colocar os nomes das propriedades nos objetos envolvidos em `["nome_aqui"]`, isso nos permite colocar nomes nas propriedades que quebrem as regras de nomeação de variáveis em JavaScript:
    - `turmas.3º ano` não é um nome de propriedade válido
    - `turmas["3º ano"]` é uma nome de propriedade válido
3. Notem que a ação que queremos executar, por receber um valor externo (a variável `alvo`) precisa ser executada dentro de uma função anônima que recebe `alvo` como argumento.

Mas temos também agora diversas vantagens:

1. A função principal `atacar` ficou incrivelmente simples e fácil de mantêr.
2. Adicionar novas habilidades requer apenas a adição de mais uma única linha para cada habilidade.
3. O programa não perde tempo fazendo comparações desnecessárias.


