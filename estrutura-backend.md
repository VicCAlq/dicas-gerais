# Estrutura do Backend

## Criação do banco e tabela

Nos nossos projetos deste trimestre trabalharemos com um backend simples, que apenas armazenará dados do progresso do jogador. Para tal, vocês devem, no `app.js` do backend de vocês, importar a biblioteca do `sqlite3` usando a linha abaixo:  

```javascript
const sql = require('sqlite3').verbose()
```

E por fim, devem criar o banco de dados e tabela que armazenará o estado de jogo de cada jogador.  

A estrutura da tabela conterá apenas 3 colunas:  

- id: Como chave primária da tabela de jogadores
- nome: Para armazenar o nome do jogador, que será utilizado como "login"
- progresso: Armazenará um texto no formato "JSON": mesmo formato do arquivo que define as ferramentas instaladas nos projetos que fazemos

Relembrando a criação do banco de dados:

```javascript
const BancoDeDados = "nome-do-jogo" // Nome do arquivo do banco de dados
// Criação do banco de dados
const db = new sql.Database(
  `./${BancoDeDados}.db`,
  (erro) => { // Função que executa que o banco é criado
    if (erro) {
      console.error(`Erro ao abrir o banco de dados "${BancoDeDados}.db":`, erro.message);
    } else {
      console.log(`Conectado ao banco de dados SQLite3 "${BancoDeDados}.db"`);
    }
  }
)
```

E por fim, criação da tabela que armazenará informações do progresso dos jogadores:

```javascript
const TabelaPrincipal = "jogadores"
db.run(
  `CREATE TABLE IF NOT EXISTS ${TabelaPrincipal} (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nome TEXT NOT NULL UNIQUE,
    progresso TEXT NOT NULL
  )`,
  // 2º argumento = função executada após termos o resultado do comando SQL
  (erro) => {
    if (erro) {
      console.error(`Erro ao criar a tabela ${TabelaPrincipal}`, erro.message);
    } else {
      console.log(`Tabela ${TabelaPrincipal} pronta!`);

      db.run(
        `INSERT INTO ${TabelaPrincipal} (nome, progresso) VALUES
          ('adm', '{}')
        `,
        (erro) => {
          if (erro) {
            console.error(`Erro ao criar inserir jogadores na tabela ${TabelaPrincipal}`, erro.message);
          } else {
            console.log(`Jogadores inseridos na tabela ${TabelaPrincipal}`);
          }
        }
      )
    }
  }
)
```

## Rota para criação de jogadores

O próximo passo é criar uma rota para criação do jogador, que receberá a informação do nome do jogador enviada pela tela título na parte de frontend da aplicação.  

No backend, esta seria a rota:

```javascript
app.post('/api/jogador/', (req, res) => {

  if (!req.body) {
    res.status(400).json({ error: erro.message });
    return
  }

  const { nome, progresso } = req.body
  
  db.all(
    `INSERT INTO ${TabelaPrincipal} (nome, progresso) VALUES (?, ?)`,
    [ nome, progresso],
    (erro, itensDaTabela) => {
      if (erro) {
        res.status(400).json({ ok: false, error: erro.message });
        return;
      }
      res.status(200).json({
        ok: true,
        message: `Jogador ${nome} criado com sucesso`,
        data: { id: this.lastID },
        id: this.lastID,
        total: itensDaTabela,
      });
    }
  )
})
```

E no frontend, esta seria a base da requisição que criaria o jogador (dentro de algum dos componentes, a seu critério):

```javascript
const [jogador, setJogador] = useState("")

async function criarJogador() {
    await fetch("http://localhost:3000/api/jogador/", {
      method: "POST",
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        nome: jogador,
        progresso: {}
      })
    })
    .then( (response) => {
       // Decida aqui o que fazer com a resposta do servidor
       // E trate os erros de acordo com o necessário
    });
}
```

## Atualização do progresso

Periodicamente, o frontend deve enviar o progresso do jogador para o backend. Para isso, precisamos estabelecer uma rota dedicada a isso:


```javascript
app.post('/api/progresso/', (req, res) => {

  if (!req.body) {
    res.status(400).json({ error: erro.message });
    return
  }

  const { nome, progresso } = req.body
  
  db.all(
    `UPDATE ${TabelaPrincipal} SET progresso = ? WHERE nome = ?`,
    [ progresso, nome],
    (erro, itensDaTabela) => {
      if (erro) {
        res.status(400).json({ ok: false, error: erro.message });
        return;
      }
      res.status(200).json({
        ok: true,
        message: `Progresso do jogador ${nome} atualizado.`,
        data: { id: this.lastID },
        id: this.lastID,
        total: itensDaTabela,
      });
    }
  )
})
```

E no frontend, esta seria a base da requisição para atualizar o progresso do jogador (dentro de algum dos componentes, a seu critério):

```javascript
const [jogador, setJogador] = useState("")
const [progresso, setProgresso] = useState({})

async function criarJogador() {
    await fetch("http://localhost:3000/api/progresso/", {
      method: "PUT",
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        nome: jogador,
        progresso: progresso,
      })
    })
    .then( (response) => {
       // Decida aqui o que fazer com a resposta do servidor
       // E trate os erros de acordo com o necessário
    });
}
```

## Carregando progresso do jogador

```javascript
app.get('/api/progresso/:nome', (req, res) => {

  if (!req.params.nome) {
    res.status(400).json({ error: erro.message });
    return
  }

  const { nome } = req.params.nome
  
  db.all(
    `SELECT * FROM ${TabelaPrincipal} WHERE nome = ?`,
    [ nome],
    (erro, itensDaTabela) => {
      if (erro) {
        res.status(400).json({ ok: false, error: erro.message });
        return;
      }
      res.status(200).json({
        ok: true,
        message: `Jogador ${nome} encontrado.`,
        data: { id: this.lastID },
        id: this.lastID,
        total: itensDaTabela,
      });
    }
  )
})
```

E no frontend, esta seria a base da requisição para atualizar o progresso do jogador (dentro de algum dos componentes, a seu critério):

```javascript
const [jogador, setJogador] = useState("")

async function criarJogador() {
    await fetch(`http://localhost:3000/api/progresso/${jogador}`, {
      method: "GET",
    })
    .then( (response) => {
       // Decida aqui o que fazer com a resposta do servidor
       // E trate os erros de acordo com o necessário
    });
}
```
