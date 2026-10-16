# Paradigmas de Linguagens de Programação: Atividade Aula 08

[← voltar ao índice do repositório](../README.md)

Exercício da pasta `07\_exercicio`, sobre **subprogramas** (capítulos 9 e 10 de
Sebesta). Para cada programa: qual é a saída e por quê.

|Item|Valor|
|-|-|
|**Aluno**|Rômulo Pardim|
|**RA**|24019338-2|
|**Referência**|SEBESTA, R. W. *Conceitos de Linguagens de Programação*, 11. ed., caps. 9 e 10|

## Resumo

|Questão|Linguagem|Saída|Conceito|
|-|-|-|-|
|1|Python|`\[1]` e `\[1, 2]`|padrão mutável avaliado uma vez|
|2|Java|`0 5`|passagem por valor da referência|
|3|Python|`\[2, 2, 2]`|fechamento com vinculação tardia|
|4|C|`3`|variável estática local|
|5|Rust|não compila|posse movida para o subprograma|
|6|Python|`UnboundLocalError`|atribuição torna o nome local|

\---

## Questão 1 (Python): parâmetro padrão mutável

```python
def adicionar(item, lista=\[]):
    lista.append(item)
    return lista

print(adicionar(1))
print(adicionar(2))
```

Saída:

```
\[1]
\[1, 2]
```

O valor padrão é avaliado **uma única vez**, quando o `def` é executado, e
fica guardado junto da função. As duas chamadas compartilham a mesma lista, e
a segunda enxerga o que a primeira acrescentou.

O padrão age como uma variável de tempo de vida igual ao da função, embora o
programador espere algo recriado a cada chamada. A forma correta é usar `None`
como padrão e criar a lista dentro do corpo.

## Questão 2 (Java): o que a passagem por valor copia

```java
static void zera(int\[] v, int n) {
    v\[0] = 0;
    n = 0;
}
// chamada com v = {5, 5} e n = 5
```

Saída:

```
0 5
```

Java passa tudo por valor, mas o valor de `v` é uma **referência**. A cópia
aponta para o mesmo vetor, então `v\[0] = 0` altera o objeto que o chamador
enxerga. Já `n` é um `int`, e a cópia local é descartada no retorno.

Não é passagem por referência: atribuir `v = new int\[]{9, 9}` dentro do método
não afetaria o chamador. O que se compartilha é o objeto, não a variável.

## Questão 3 (Python): fechamento com vinculação tardia

```python
fs = \[lambda: i for i in range(3)]
print(\[f() for f in fs])
```

Saída:

```
\[2, 2, 2]
```

As três funções capturam a **variável** `i`, não o valor que ela tinha na
criação. Quando são chamadas, a compreensão já terminou e `i` vale 2, então as
três devolvem 2.

Correção usual: `lambda i=i: i`, que transforma a captura em um parâmetro com
padrão avaliado na definição.

## Questão 4 (C): variável estática local

```c
int contador(void) {
    static int n = 0;
    return ++n;
}
// contador(); contador(); printf("%d\\n", contador());
```

Saída:

```
3
```

`static` dentro de uma função dá escopo local e tempo de vida igual ao do
programa. A variável é inicializada uma vez e o valor sobrevive entre
chamadas, então as três chamadas devolvem 1, 2 e 3.

É o exemplo mais direto de escopo e tempo de vida serem eixos independentes:
o menor escopo possível com o maior tempo de vida possível.

## Questão 5 (Rust): posse transferida na chamada

```rust
fn dobra(v: Vec<i32>) -> Vec<i32> { ... }

let v = vec!\[1, 2, 3];
let d = dobra(v);
println!("{:?} {:?}", v, d);
```

Não compila, com erro `E0382`.

O parâmetro é declarado como `Vec<i32>` por valor, então a chamada **move** a
posse do vetor para dentro da função. Depois do `let d = dobra(v)`, a variável
`v` não é mais válida, e usá-la no `println!` é o erro.

Correção: receber por empréstimo, com `fn dobra(v: \&\[i32])` e chamar
`dobra(\&v)`. A passagem de parâmetro em Rust também decide quem é o dono,
não apenas o que é copiado.

## Questão 6 (Python): atribuição define o escopo

```python
total = 0

def adiciona(x):
    total = total + x
    return total
```

Saída:

```
UnboundLocalError: cannot access local variable 'total' where it is not associated with a value
```

Python decide o escopo de um nome na **compilação** do corpo: como `total`
aparece do lado esquerdo de uma atribuição, ele vira local em toda a função.
A leitura do lado direito acontece antes de qualquer atribuição, e a variável
local ainda não tem valor.

Correção: declarar `global total`, ou receber e devolver o valor em vez de
usar estado global, que é a opção preferível.

\---

## Conclusão

Os seis casos tratam da mesma pergunta: **o que o subprograma compartilha com
quem o chamou**.

* **Nada** (questões 2 em relação a `n`, e 6): a cópia local morre no retorno;
* **O objeto, não a variável** (questão 2 em relação a `v`): a referência é
copiada, o alvo é o mesmo;
* **A posse** (questão 5): Rust torna a transferência explícita e verificada;
* **Estado que sobrevive à chamada** (questões 1, 3 e 4): padrão mutável,
fechamento e `static` guardam valor entre chamadas, e é aí que mora a
surpresa.

As questões 1, 3 e 4 mostram que tempo de vida maior que o da chamada é um
recurso legítimo, só que fácil de acionar sem querer.

## Verificação

A questão 2 foi executada localmente (OpenJDK 21.0.11) e produziu exatamente
`0 5`. As questões 1, 3, 4, 5 e 6 não puderam ser executadas, por falta de
Python, gcc e Rust nesta máquina, e as saídas foram deduzidas da semântica de
cada linguagem.

