# Lógica de renderização

1. Separar a parte lógica (grid, personagens, peças, etc) da parte visual (React)
2. Parte visual = arquivos JSX que tem os componentes React
3. Parte lógica = Objetos e classes em javascript comum

Exemplo da parte lógica:
```javascript
export default class Grid {
  constructor(altura, largura) {
    this.altura = altura
    this.largura = largura
    const grid = []

    // obs 1
    for(let linhas = 0; linhas < altura; linhas ++) {
      let nova_linha = []
      for(let colunas = 0; colunas < largura; colunas++) {
        nova_linha.push(0)
      }
      grid.push(nova_linha)
    }

    this.grid = grid
  }

  transformarCasa(posVertical, posHorizontal, valor) {
    this.grid[posVertical][posHorizontal] = valor
  }

  mover(direcao, posVertical, posHorizontal, distancia) {
    let entidadeOriginal = this.grid[posVertical][posHorizontal]

    if (direcao.toLowerCase() === "c") {
      this.grid[posVertical - distancia][posHorizontal] = entidadeOriginal
      this.grid[posVertical][posHorizontal] = 0

    } else if (direcao.toLowerCase() === "b") {
      this.grid[posVertical + distancia][posHorizontal] = entidadeOriginal
      this.grid[posVertical][posHorizontal] = 0

    } else if (direcao.toLowerCase() === "e") {
      this.grid[posVertical][posHorizontal - distancia] = entidadeOriginal
      this.grid[posVertical][posHorizontal] = 0

    } else if (direcao.toLowerCase() === "d") {
      this.grid[posVertical][posHorizontal + distancia] = entidadeOriginal
      this.grid[posVertical][posHorizontal] = 0
    }
  }
}
```
No script acima, fazemos uso de uma classe comum para criar objetos do tipo `Grid`.
Utilizamos o laço de repetição marcado na `obs 1` para, a partir da altura e largura
informados na criação do elemento, criarmos uma matriz de "X" altura por "Y" largura.
Este grid é então armazenado na propriedade `grid` do objeto.  

A matriz é criada com todas as suas "casas" iniciadas com o valor `0`. A ideia
é que para cada elemento que possa ocupar uma casa, seja atribuído um valor
numérico distinto, de modo a simplificar verificação de posições, e transformação
de um valor em outro (para modificar o conteúdo da casa).  

Movimentação pelo grid é feito a partir da manipulação de índices deste grid, onde:  
- `grid[0][0]` representa a posição do topo a esquerda  
- `grid[altura - 1][largura - 1]` representa a posição do fundo a direita  
Podemos somar e subtrair destes dois índices para simular o "movimento" de uma peça
pelas casas do grid. Podem conferir um exemplo de como implementar isso no método
`mover` exemplificado acima.

No frontend, este objeto `Grid` é enviado para um componente responsável por
ler o conteúdo deste componente, e exibir na tela a visualização adequada:

```jsx
import { View, Text } from 'react-native';

export default function GridJogo({grid}) {
  
  return(<View>
    <Text>Grid Total</Text>
    {grid.map((linha, idxLinha) => {
      return <View>
        <Text>Linha</Text>
        {linha.map((coluna, idxColuna) => {
          {coluna === 0
            ? <View><Text>Casa vazia</Text></View>
            : <View><Text>Casa preenchida</Text></View>
          }
        })}
      </View> 
    })}
  </View>)
}
```
Componentes do React e React Native atualizam seu conteúdo por conta própria, sempre
que o valor que recebem como argumento (`grid` no exemplo acima) é modificado.
Se o `grid` for armazenado em um `useState` do elemento que chama este `GridJogo`,
toda vez que for usado `setState` para modificar o `grid`, `GridJogo` se atualiza
com os valores novos.  

Com isso separamos nossa lógica de jogo da nossa lógica de exibição.

