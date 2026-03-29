# ALU - Unidade Lógica e Aritmética

Este projeto implementa uma ALU (Arithmetic Logic Unit) de 8 bits desenvolvida no simulador Digital, construída de forma incremental a partir de componentes de 1 bit. Os arquivos `.dig` contêm os circuitos simuláveis de cada componente.

---

## Arquivo Principal

A ALU completa, com todos os componentes conectados e seleção de operação, está no arquivo `ALU.dig`. Os demais arquivos são os subcomponentes desenvolvidos individualmente ao longo do processo.

---

## Estrutura dos Arquivos

| Arquivo                         | Descrição                                                       |
| ------------------------------- | --------------------------------------------------------------- |
| `ALU.dig`                       | **ALU completa com todos os componentes e seleção de operação** |
| `Soma_1bit.dig`                 | Somador completo de 1 bit                                       |
| `Soma_4bit.dig`                 | Somador de 4 bits                                               |
| `Soma_8bit.dig`                 | Somador de 8 bits                                               |
| `Soma_16bit.dig`                | Somador de 16 bits (usado na multiplicação)                     |
| `Subtracao_1bit.dig`            | Subtrator completo de 1 bit                                     |
| `Subtracao_4bit.dig`            | Subtrator de 4 bits                                             |
| `Subtracao_8bit.dig`            | Subtrator de 8 bits                                             |
| `Left_Shift.dig`                | Deslocamento de bits à esquerda                                 |
| `Right_Shift.dig`               | Deslocamento de bits à direita                                  |
| `Multiplexador_8bit.dig`        | Multiplexador de 8 bits                                         |
| `NAND_8bit.dig`                 | Operação NAND de 8 bits                                         |
| `XOR_8bit.dig`                  | Operação XOR de 8 bits                                          |
| `Multiplicacao-8bit.dig`        | Multiplicador de 8 bits (versão final)                          |
| `Multiplicacao_8bit-Errado.dig` | Primeira tentativa do multiplicador (abordagem sem shifter)     |

---

## Somador

### Somador de 1 bit

O ponto de partida foi o somador completo de 1 bit, que recebe três entradas: `A`, `B` e `Cin` (carry de entrada), e produz duas saídas: `S` (soma) e `Cout` (carry de saída).

A lógica foi derivada diretamente da tabela verdade:

| A   | B   | Cin | S   | Cout |
| --- | --- | --- | --- | ---- |
| 0   | 0   | 0   | 0   | 0    |
| 0   | 0   | 1   | 1   | 0    |
| 0   | 1   | 0   | 1   | 0    |
| 0   | 1   | 1   | 0   | 1    |
| 1   | 0   | 0   | 1   | 0    |
| 1   | 0   | 1   | 0   | 1    |
| 1   | 1   | 0   | 0   | 1    |
| 1   | 1   | 1   | 1   | 1    |

A partir da tabela, foram extraídas as expressões booleanas:

```
S    = A XOR B XOR Cin
Cout = (A AND B) OR (A AND Cin) OR (B AND Cin)
```

O circuito foi implementado com uma porta XOR de 3 entradas para a soma, e três portas AND combinadas por uma porta OR de 3 entradas para o carry de saída.

### Somador de 4 bits

Com o somador de 1 bit funcionando, o somador de 4 bits foi construído encadeando quatro instâncias dele. O `Cout` de cada estágio é conectado ao `Cin` do próximo, formando um ripple carry adder. O carry inicial (`Cin` do bit menos significativo) é fixado em 0.

### Somador de 8 bits

Seguindo o mesmo princípio, dois somadores de 4 bits foram encadeados para formar o somador de 8 bits. O `Cout` do somador dos 4 bits menos significativos alimenta o `Cin` do somador dos 4 bits mais significativos.

---

## Subtrator

### Subtrator de 1 bit

O subtrator de 1 bit recebe `A`, `B` e `Cin` (borrow de entrada) e produz `S` (diferença) e `Cout` (borrow de saída).

A tabela verdade foi montada para a operação `A - B - Cin`:

| A   | B   | Cin | S   | Cout |
| --- | --- | --- | --- | ---- |
| 0   | 0   | 0   | 0   | 0    |
| 0   | 0   | 1   | 1   | 1    |
| 0   | 1   | 0   | 1   | 1    |
| 0   | 1   | 1   | 0   | 1    |
| 1   | 0   | 0   | 1   | 0    |
| 1   | 0   | 1   | 0   | 0    |
| 1   | 1   | 0   | 0   | 0    |
| 1   | 1   | 1   | 1   | 1    |

As expressões booleanas resultantes foram:

```
S    = A XOR B XOR Cin
Cout = (NOT A AND B) OR (NOT A AND Cin) OR (B AND Cin)
```

Note que a saída `S` tem a mesma expressão do somador, mas o `Cout` (borrow) difere: as condições de estouro agora envolvem `NOT A`, pois o borrow ocorre quando se tenta subtrair um valor maior do que `A` permite.

### Subtrator de 4 e 8 bits

O mesmo processo de encadeamento usado no somador foi aplicado ao subtrator: quatro subtratores de 1 bit formam o de 4 bits, e dois subtratores de 4 bits formam o de 8 bits, propagando o borrow entre os estágios.

---

## Multiplicador

### Primeira Abordagem (sem shifter)

A ideia inicial foi implementar a multiplicação sem utilizar deslocadores de bits. A intenção era aproveitar o `Cout` do somador para propagar automaticamente os bits para posições mais significativas, eliminando a necessidade de shifters explícitos.

Porém, ao desenvolver o circuito, ficou claro que controlar o fluxo dos carries parciais e alinhá-los corretamente para cada produto parcial tornava a lógica progressivamente mais complexa e difícil de depurar. Essa tentativa está preservada no arquivo `Multiplicacao_8bit-Errado.dig`.

### Abordagem Final (com shifter e somador de 16 bits)

A solução adotada foi baseada no algoritmo clássico de multiplicação por somas e deslocamentos:

1. Para cada bit do multiplicador, o multiplicando é deslocado à esquerda pelo número de posições correspondente ao peso do bit, usando o componente `Left_Shift.dig`.
2. Os produtos parciais resultantes são somados usando um somador de 16 bits, construído encadeando dois somadores de 8 bits.
3. A saída do multiplicador é de 16 bits, mas apenas os 8 bits mais significativos são utilizados como resultado final, correspondendo ao produto de dois operandos de 8 bits dentro da faixa representável.

Essa abordagem torna o circuito modular e reutiliza componentes já validados nas etapas anteriores.

---

## Outros Componentes

- **Left_Shift / Right_Shift**: deslocadores de bits usados na multiplicação e disponíveis como operações independentes da ALU.
- **Multiplexador_8bit**: seleciona entre múltiplas entradas de 8 bits, utilizado para rotear operandos ou resultados dentro da ALU.
- **NAND_8bit**: aplica a operação NAND bit a bit em dois operandos de 8 bits.
- **XOR_8bit**: aplica a operação XOR bit a bit em dois operandos de 8 bits.

---

## Metodologia Geral

O desenvolvimento seguiu uma abordagem bottom-up: cada componente mais complexo foi construído a partir de instâncias do componente imediatamente anterior, validadas individualmente. Isso garantiu que os erros pudessem ser isolados no menor nível possível antes de serem integrados em componentes maiores.

## Vídeo de Demonstração

Clique [aqui](https://youtu.be/138wBpV3CdY) para acessar o vídeo de demonstração e explicação do funcionamento da ALU.
