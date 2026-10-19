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

  function iniciar() {
    setIniciado(!iniciado)
  }

  return (
    <View>
      <TelaTitulo jogador={jogador} setJogador={setJogador}, setIniciado={iniciar} />
      {iniciado !== false ?
        <TelaJogo jogador={jogador}/> :
        <View>
          <Text>
            Jogo não iniciado
          </Text>
        </View>
      }
      <StatusBar style="auto" />
    </View>
  );
}
```

- TelaTitulo.jsx
```javascript
export default function TelaTitulo({ jogador, setJogador, setIniciado }) {

  return (<View>
    <Text> Tela inicial </Text>
    <Text> {jogador !== "" ? jogador : "Jogador não informado"} </Text>
    <TextInput
      placeholder="Nome do Jogador"
      onChangeText={setJogador}
      value={jogador}
    />
    <Pressable onPress={setIniciado} >
      <Text>Iniciar</Text>
    </Pressable>
  </View>)
}
```

- TelaJogo.jsx
```javascript
export default function TelaJogo({jogador}) {
  
  return(<View>
    <Text>
      Jogo iniciado com o jogador {jogador}
    </Text>
  </View>)
}
```

Notem que no exemplo acima, precisamos criar todos os valores existentes dentro do componente `App`, e então precisamos passar cada variável e função para os componentes que vão fazer uso destes valores. Em um jogo como o que vocês farão, vocês poderão chegar a utilizar centenas de estados diferentes, e podem se perder na hora de passar as variáveis para os componentes adequados.

## Solução

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
  }))
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
  setIniciado: () => set((estado) => ({ iniciado: !estado.iniciado }))
}))

export default useEstado
```

E os componentes agora não precisam se preocupar em passar variáveis de uns para os outros, basta importarem os elementos desejados da função `useEstado()` criada acima.

- App.jsx
```javascript
export default function App() {

  const { iniciado } = useEstado()

  return (
    <View>
      <TelaTitulo/>
      {iniciado !== false ?
        <TelaJogo/> :
        <View> <Text> Jogo não iniciado </Text> </View>
      }
      <StatusBar style="auto" />
    </View>
  );
}
```

- TelaTitulo.jsx
```javascript
export default function TelaTitulo({}) {

  const { jogador, setJogador, setIniciado, } = useEstado()

  return (<View>
    <Text> Tela inicial </Text>
    <Text>
      {jogador !== "" ? jogador : "Jogador não informado"}
    </Text>
    <TextInput
      placeholder="Nome do Jogador"
      onChangeText={setJogador}
      value={jogador}
    />
    <Pressable onPress={setIniciado} >
      <Text>Iniciar</Text>
    </Pressable>
  </View>)
}
```

- TelaJogo.jsx
```javascript
export default function TelaJogo({jogador}) {

  const {jogador} = useEstado()

  return(<View>
    <Text>
      Jogo iniciado com o jogador {jogador}
    </Text>
  </View>)
}
```

Para aprender mais, consultem a documentação da biblioteca: https://zustand.docs.pmnd.rs/
