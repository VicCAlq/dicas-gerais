#  Banco de dados para salvar o jogo

Ao desenvolvermos nossos jogos, vamos precisar salvar as informações de quem estiver jogando, de modo que o jogo possa ser interrompido e retomado do ponto onde foi encerrado anteriormente.

Para isso utilizaremos a biblioteca `idb`.

## Como instalar

Dentro da pasta principal do projeto (onde tem o arquivo `package.json`), rode o comando `npm install idb`

## Como usar

Para nossos jogos, precisamos apenas de duas funções para usar a biblioteca: `SalvarJogo` e `CarregarJogo`.

Dentro de um arquivo chamado `JogosSalvos.js`, coloquem o código a seguir:

```javascript
import { openDB } from 'idb';

const DB_NOME = 'nome_do_seu_jogo_aqui';
const DB_VERSAO = 1;

function conectarDB() {
  return openDB(DB_NOME, DB_VERSAO, {
    upgrade(db) {
      if (!db.objectStoreNames.contains('saves')) {
        const saves = db.createObjectStore('saves', {
          keyPath: 'id',
          autoIncrement: true
        })
        fontes.createIndex('jogador', 'jogador', { unique: false })
        fontes.createIndex('estado', 'estado', { unique: false })
      }
    }
  })
}

/**
  * Recebe o estado de jogo para ser salvo na aplicação. Contém
  * as propriedades `jogador` contendo o nome do jogador, e
  * a propriedade `estado` contendo o estado de jogo administrado
  * pela biblioteca `zustand` e convertido em texto JSON usando
  * a função `JSON.stringify()`
  * @param {{jogador: string, estado: string}} save
  */
async function SalvarJogo(save) {
  const db = await conectarDB();
  return db.add('saves', save)
}

/**
  * Recebe o nome do jogador que terá seu save carregado.
  * O Save será retornado como um texto JSON que deve ser convertido
  * para objeto JavaScript usando a função `JSON.parse()`
  * @param {string} jogador
  */
async function CarregarJogo(jogador) {
  const db = await conectarDB();
  const save = db.getAllFromIndex('saves', 'jogador', jogador)
  return JSON.parse(save)
}

export { conectarDB, SalvarJogo, CarregarJogo }
```

E no jogo de vocês, vocês podem importar as funções `SalvarJogo` e `CarregarJogo` deste arquivo para salvar e carregar os jogos de vocês.
