# Estado global de jogo

## Diferença entre useState e Estado Global

Até o presente momento, aprendemos a modificar valores internos de um componente utilizando `useState`, e quando queremos que um elemento tenha seu valor modificado por alguma ação em outro elemento, precisamos passar os valores deste `state` para o componente responsável por esta ação.

Este formato de passar responsabilidades de modificar valores de um componente para outro se chama `prop drilling`, e com a expansão dos comportamentos que ocorrem no aplicativo, fazer isso se torna cada vez mais complexo.

Para evitar este problema, vamos utilizar a biblioteca `Zustand` para criar um **Estado Global**, onde colocaremos todas as variáveis e funções responsáveis por modificar estas variáveis, e podemos acessar de qualquer local da aplicação.

## Exemplo e Problema

Considerem os componentes `App`, `TelaTitulo` e `TelaJogo` abaixo, utilizando `useState` simples:

- App.jsx
```javascript
export default function App() {

  const [jogador, setJogador] = useState(false)
  const [iniciado, setIniciado] = useState(false)
  const [contador, setContador] = useState(false)

  function iniciar() { setIniciado(!iniciado) }

  function aumentarContador(numero) { setContador(contador + numero) }

  function reduzirContador(numero) { setContador(contador - numero) }

  return (
    <div>
      <TelaTitulo
        jogador={jogador}
        setJogador={setJogador}
        setIniciado={iniciar}
        aumentarContador={aumentarContador}
        reduzirContador={reduzirContador}
      />
      {iniciado !== false ?
        <TelaJogo
          jogador={jogador} 
          contador={contador}
        /> :
        <div>
          <p> Jogo não iniciado </p>
        </div>
      }
      <StatusBar style="auto" />
    </div>
  );
}
```

- TelaTitulo.jsx
```javascript
export default function TelaTitulo({ 
  jogador,
  setJogador,
  setIniciado,
  aumentarContador,
  reduzirContador
}) {

  return (<div>
    <p> Tela inicial </p>
    <p> {jogador !== "" ? jogador : "Jogador não informado"} </p>
    <input
      placeholder="Nome do Jogador"
      onChange={(e) => setJogador(e.target.value)}
      value={jogador}
    />
    <button onClick={setIniciado} >
      <p>Iniciar</p>
    </button>
    <button onClick={() => aumentarContador(3)} >
      <p>Aumentar Contador</p>
    </button>
    <button onClick={() => reduzirContador(2)} >
      <p>Reduzir Contador</p>
    </button>
  </div>)
}
```

- TelaJogo.jsx
```javascript
export default function TelaJogo({jogador, contador}) {

  return(<div>
    <p> Jogo iniciado com o jogador {jogador} </p>
    <p> Valor do contador: {contador} </p>
  </div>)
}
```

Notem que no exemplo acima, precisamos criar todos os valores existentes dentro do componente `App`, e então precisamos passar cada variável e função para os componentes que vão fazer uso destes valores. Em um jogo como o que vocês farão, vocês poderão chegar a utilizar centenas de estados diferentes, e podem se perder na hora de passar as variáveis para os componentes adequados.

## Solução

Vamos para isso usar uma biblioteca chamada `Zustand`. Instale ela rodando o comando `npm install zustand` com o terminal na pasta correta do projeto (a que tem o arquivo `package.json`)  
Ao utilizar o `Zustand`, criamos todos os valores e suas funções modificadoras em um arquivo a parte, cujo nome e elemento exportado devem começar com `use`:

- useEstado.jsx
```javascript
import { create } from "zustand"

const useEstado = create((set) => ({
  // equivalente a variável "jogador" do estado
  // const [jogador, setJogador] = useState("Chapolin Colorado")
  jogador: "Chapolin Colorado",
  // setJogador vai ser uma função anônima que recebe o novo 
  // valor "nome", e atribui esse "nome" para a propriedade "jogador".
  // É equivalente a função "setJogador" do estado
  // const [jogador, setJogador] = useState("Chapolin Colorado")
  setJogador: (nome) => set(() => ({
    jogador: nome
  })),

  // equivalente a variável "iniciado" do estado
  // const [iniciado, setIniciado] = useState(false)
  iniciado: false,
  // setIniciado vai ser uma função anônima que neste caso não
  // recebe um argumento novo, mas vai enxergar o próprio estado
  // global e modificar o valor de "iniciado" para o seu oposto.
  // É equivalente a função "setIniciado" do estado
  // const [iniciado, setIniciado] = useState(false)
  setIniciado: () => set((estado) => ({
    iniciado: !estado.iniciado
  })),
  // equivalente a variável "contador" do estado
  // const [contador, setContador] = useState(0)
  contador: 0,
  // aumentarContador vai ser uma função anônima que recebe
  // um número a ser adicionado a variável contador, recebendo
  // o valor anterior do contador e acrescentando o número informado.
  aumentarContador: (numero) => set((estado) => ({
    contador: estado.contador + numero
  })),
  // reduzirContador é o mesmo que "aumentarContador", mas vai
  // reduzir o número informado do contador ao invés de aumentar.
  reduzirContador: (numero) => set((estado) => ({
    contador: estado.contador - numero
  })),
}))

export default useEstado
```

Ignorando os comentários explicativos, o código é bem pequeno:
```javascript
import { create } from "zustand"

const useEstado = create((set) => ({
  jogador: "Chapolin Colorado",
  setJogador: (nome) => set(() => ({ jogador: nome })),
  iniciado: false,
  setIniciado: () => set((estado) => ({ iniciado: !estado.iniciado })),
  contador: 0,
  aumentarContador: (numero) => set((estado) => ({
    contador: estado.contador + numero
  })),
  reduzirContador: (numero) => set((estado) => ({
    contador: estado.contador - numero
  })),
}))

export default useEstado
```

E os componentes agora não precisam se preocupar em passar variáveis de uns para os outros, basta importarem os elementos desejados da função `useEstado()` criada acima.

- App.jsx
```javascript
export default function App() {

  const { iniciado } = useEstado()

  return (
    <div>
      <TelaTitulo/>
      {iniciado !== false ?
        <TelaJogo/> :
        <div> <p> Jogo não iniciado </p> </div>
      }
      <StatusBar style="auto" />
    </div>
  );
}
```

- TelaTitulo.jsx
```javascript
export default function TelaTitulo({}) {

  const {
    jogador,
    setJogador,
    setIniciado,
    aumentarContador,
    reduzirContador
  } = useEstado()

  return (<div>
    <p> Tela inicial </p>
    <p>
      {jogador !== "" ? jogador : "Jogador não informado"}
    </p>
    <input
      placeholder="Nome do Jogador"
      onChange={(e) => setJogador(e.target.value)}
      value={jogador}
    />
    <button onClick={() => setIniciado()} >
      <p>Iniciar</p>
    </button>
    <button onClick={() => aumentarContador(3)} >
      <p>Aumentar Contador</p>
    </button>
    <button onClick={() => reduzirContador(2)} >
      <p>Reduzir Contador</p>
    </button>
  </div>)
}
```

- TelaJogo.jsx
```javascript
export default function TelaJogo({jogador}) {

  const {jogador} = useEstado()

  return(<div>
    <p> Jogo iniciado com o jogador {jogador} </p>
    <p> Valor do contador: {contador} </p>
  </div>)
}
```

Para aprender mais, consultem a documentação da biblioteca: https://zustand.docs.pmnd.rs/
