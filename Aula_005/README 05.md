# Paradigmas de Linguagens de Programação: Atividade Aula 05

[← voltar ao índice do repositório](../README.md)

Atividade: **exercícios autorais sobre Nomes, Vinculações e Escopo**, baseados
no capítulo 5 de Sebesta.

A lista do professor traz 27 exercícios ([exercicios.pdf](exercicios.pdf)). A
tarefa pede 7 deles, em ordem livre, desde que **não sejam consecutivos**.

| Item | Valor |
|---|---|
| **Aluno** | Leonardo Camilotti Moreno |
| **RA** | 24015988-2 |
| **Exercícios escolhidos** | 1, 3, 6, 9, 13, 17, 20 |
| **Referência** | SEBESTA, R. W. *Conceitos de Linguagens de Programação*, 11. ed., cap. 5 |

## Estrutura do repositório

```
.
├── README.md         # este documento (respostas)
└── exercicios.pdf    # enunciado dos 27 exercícios
```

## Critério de escolha

Nenhum par escolhido é consecutivo: o menor intervalo entre dois números é 2.

```
1 ... 3 ... 6 ... 9 ... 13 ... 17 ... 20
 +2    +3    +3    +4     +4     +3
```

A seleção também cobre os cinco tipos de tarefa da lista e os três eixos da
aula:

| Exercício | Tema | Tipo de tarefa | Eixo |
|---|---|---|---|
| 1 | Linguagens imperativas e von Neumann | aplicar a um exemplo | fundamento |
| 3 | Case sensitive | construir exemplo mínimo | nomes |
| 6 | Atributos das variáveis | aplicar a um exemplo | variáveis |
| 9 | Tempos de vinculação | comparar duas situações | vinculações |
| 13 | Vinculação de tipos dinâmica | construir exemplo mínimo | vinculações |
| 17 | Variáveis dinâmicas do heap | analisar erro conceitual | armazenamento |
| 20 | Escopo estático | explicar para um colega | escopo |

---

## 1. Linguagens imperativas e arquitetura de von Neumann

> Aplique o conceito a um exemplo autoral pequeno. Descreva o contexto, a
> decisão tomada e a consequência esperada.

### Contexto

Preciso somar os valores de um vetor com 1 milhão de inteiros e devolver o
total. Duas implementações resolvem o problema:

```java
// A: laço com acumulador
long somaIterativa(int[] v) {
    long total = 0;
    for (int i = 0; i < v.length; i++) {
        total += v[i];
    }
    return total;
}

// B: recursão
long somaRecursiva(int[] v, int i) {
    if (i == v.length) return 0;
    return v[i] + somaRecursiva(v, i + 1);
}
```

### Decisão tomada

Escolhi a versão A, com laço e variável acumuladora.

### Consequência esperada

A versão A espelha a arquitetura de von Neumann quase linha a linha. A variável
`total` é o nome de uma célula de memória, o `+=` é uma transferência entre
memória e processador, e o laço reproduz o ciclo de busca, decodificação e
execução sobre instruções guardadas em posições vizinhas. O compilador
consegue manter `total` e `i` em registradores, e o vetor é percorrido de forma
sequencial, o que favorece a cache.

A versão B falha por 1 milhão de níveis de recursão, porque cada chamada empilha
um registro de ativação. Em Java isso estoura a pilha e lança
`StackOverflowError`.

O ponto conceitual: a iteração é eficiente em linguagens imperativas justamente
porque o hardware é von Neumann. O gargalo de von Neumann, a limitação do canal
entre memória e processador, é o que torna a atribuição a operação central
desse paradigma. Uma linguagem funcional pura precisaria de otimização de
chamada de cauda para competir aqui.

---

## 3. Case sensitive

> Construa um exemplo mínimo que demonstre o conceito. Identifique a situação
> observada, a aplicação do conceito e a conclusão esperada.

### Exemplo mínimo

```java
int contadorTotal = 0;
contadorTotal = contadorTotal + 1;
ContadorTotal = ContadorTotal + 1;  // erro de compilação
```

Em Python o mesmo problema aparece só na execução:

```python
contador_total = 0
print(Contador_Total)  # NameError: name 'Contador_Total' is not defined
```

### Situação observada

`contadorTotal` e `ContadorTotal` diferem em um único caractere, o `C` inicial.
Para o leitor humano são obviamente o mesmo nome; para o compilador são dois
nomes distintos. Java acusa `cannot find symbol` e Python levanta `NameError`.

### Aplicação do conceito

Uma linguagem é case sensitive quando maiúsculas e minúsculas geram lexemas
diferentes. O analisador léxico compara os caracteres um a um, sem normalizar
nada, então `total`, `Total` e `TOTAL` são três identificadores independentes.
C, C++, Java, Python, JavaScript e Go se comportam assim. Pascal, Ada e SQL não
distinguem, e nesses casos os três nomes acima seriam o mesmo.

### Conclusão esperada

Case sensitivity prejudica a legibilidade, que é exatamente o critério que
Sebesta usa para avaliar o projeto de nomes. O erro é fácil de cometer e difícil
de enxergar, porque a diferença é visualmente mínima. Em Java o custo é baixo,
já que o compilador acusa antes de rodar. Em Python o custo é bem maior: o
`NameError` só aparece quando aquela linha executa, o que pode acontecer só em
produção, num ramo raro do código.

O problema não se resolve mudando a linguagem, e sim adotando uma convenção
única de escrita, como `camelCase` em Java e `snake_case` em Python, para que
nunca existam dois nomes que só diferem em capitalização.

---

## 6. Atributos das variáveis

> Aplique o conceito a um exemplo autoral pequeno. Descreva o contexto, a
> decisão tomada e a consequência esperada.

### Contexto

Uma classe calcula o saldo de uma conta a partir de uma lista de lançamentos.
A questão é onde declarar a variável `saldo`:

```java
public class Conta {
    private double saldo;              // opção A: campo de instância

    public double calcular(List<Double> lancamentos) {
        double saldo = 0;              // opção B: variável local
        for (double l : lancamentos) {
            saldo += l;
        }
        return saldo;
    }
}
```

### Decisão tomada

Escolhi a opção B, variável local ao método.

### Consequência esperada

Sebesta descreve seis atributos de uma variável. A decisão muda quatro deles:

| Atributo | Opção A (campo) | Opção B (local) |
|---|---|---|
| **Nome** | `saldo` | `saldo` |
| **Endereço** | dentro do objeto, no heap | na pilha, no registro de ativação |
| **Tipo** | `double` | `double` |
| **Valor** | persiste entre chamadas | recomeça em 0 a cada chamada |
| **Tempo de vida** | enquanto o objeto existir | enquanto o método executar |
| **Escopo** | toda a classe | apenas o método |

A consequência prática é que a opção A acumularia o saldo entre chamadas: chamar
`calcular` duas vezes com a mesma lista devolveria o dobro na segunda vez. A
opção B devolve sempre o mesmo resultado para a mesma entrada, porque o valor é
reinicializado junto com o tempo de vida da variável.

Vale notar que nome e endereço não são a mesma coisa. O nome `saldo` aparece dos
dois lados da atribuição `saldo += l`, mas com sentidos diferentes: à esquerda
ele vale como endereço, o l-value, e à direita como valor, o r-value. É
justamente por isso que endereço e valor precisam ser atributos separados.

---

## 9. Tempos de vinculação

> Compare duas situações, uma que considere o conceito e outra que o ignore.
> Aponte uma diferença observável na interpretação ou na decisão.

O sistema precisa aplicar uma alíquota de imposto de 10% sobre cada venda. Duas
equipes resolvem, e a diferença está em **quando** o valor 10 se liga ao nome
`ALIQUOTA`.

### Situação A: considera o tempo de vinculação

```java
public static final int ALIQUOTA = 10;   // vinculação em tempo de compilação

double imposto(double valor) {
    return valor * ALIQUOTA / 100;
}
```

A equipe escolheu conscientemente vincular em tempo de compilação, por entender
que a alíquota é estável e que o ganho de desempenho compensa.

### Situação B: ignora o tempo de vinculação

```java
double imposto(double valor) {
    int aliquota = Integer.parseInt(
        Files.readString(Path.of("config.txt")).trim());  // vinculação em execução
    return valor * aliquota / 100;
}
```

A equipe não pensou no assunto e leu a alíquota do arquivo dentro do método, a
cada chamada.

### Diferença observável

| | Situação A | Situação B |
|---|---|---|
| **Momento da vinculação** | compilação | execução, a cada chamada |
| **Mudar a alíquota** | exige recompilar e publicar de novo | basta editar `config.txt` |
| **Custo por chamada** | nenhum, o compilador substitui por `valor * 10 / 100` | uma leitura de disco e um parse |
| **Erro de configuração** | impossível, o valor está no código | `NumberFormatException` em execução |

A diferença mais visível é de desempenho. Em A o compilador faz *constant
folding* e a operação some; em B cada cálculo toca o disco, o que num laço sobre
milhares de vendas domina o tempo total.

A lição não é que A seja melhor que B. Vinculação mais cedo dá eficiência,
vinculação mais tarde dá flexibilidade, e é exatamente esse o trade-off que
Sebesta apresenta. O erro da situação B não foi escolher execução, foi **não
escolher**: ninguém decidiu, e a leitura acabou dentro do método por acidente.
Se a alíquota realmente muda, a vinculação correta seria em tempo de carga, lendo
o arquivo uma vez na inicialização.

---

## 13. Vinculação de Tipos Dinâmica

> Construa um exemplo mínimo que demonstre o conceito. Identifique a situação
> observada, a aplicação do conceito e a conclusão esperada.

### Exemplo mínimo

```python
dado = 10
print(dado / 4)    # 2.5
print(type(dado))  # <class 'int'>

dado = "10"
print(type(dado))  # <class 'str'>
print(dado / 4)    # TypeError: unsupported operand type(s) for /: 'str' and 'int'
```

### Situação observada

A mesma variável `dado` é inteiro nas três primeiras linhas e string nas três
últimas. Nenhuma declaração foi escrita e nenhum erro foi acusado na mudança. O
programa roda normalmente até a última linha, onde falha.

### Aplicação do conceito

Na vinculação dinâmica de tipos, o tipo não pertence à variável, e sim ao valor
que ela guarda no momento. A vinculação acontece na atribuição e é desfeita na
atribuição seguinte. Por isso `dado` não "é" um inteiro: ela **referencia** um
inteiro até a linha 4 e uma string a partir dali.

A consequência de implementação é que o tipo precisa ser guardado junto com o
valor, em tempo de execução, e cada operação precisa consultá-lo. É isso que a
mensagem de erro revela: o operador `/` verificou os tipos dos operandos em
execução e não achou uma definição para `str` dividido por `int`.

### Conclusão esperada

A vinculação dinâmica troca verificação por flexibilidade, e o preço aparece em
três frentes:

- **Detecção tardia.** Em Java, `dado = "10"` depois de `int dado` nem compila.
  Em Python o erro só surge quando aquela linha executa, o que pode levar meses
  se o ramo for raro.
- **Custo de execução.** Cada operação carrega verificação de tipo, e a variável
  vira um descritor com tipo e ponteiro, em vez de um valor cru.
- **Legibilidade.** Sem a declaração, ler o código não basta para saber o tipo,
  é preciso rastrear todas as atribuições.

Em troca, o mesmo código serve para vários tipos sem duplicação, o que é a base
do *duck typing*. Anotações de tipo como `dado: int = 10` mitigam o problema,
mas não mudam a semântica: elas servem a ferramentas externas como o mypy e o
interpretador continua ignorando.

---

## 17. Variáveis dinâmicas do heap

> Analise o erro conceitual: alguém tratou o tema como uma etapa isolada, sem
> relação com os objetivos ou com os demais conceitos. Explique o problema e
> reescreva a conclusão.

### A conclusão equivocada

> "Variáveis dinâmicas do heap são as criadas com `new` em tempo de execução.
> Basta lembrar que ficam no heap, e não na pilha. É o terceiro tipo de
> alocação da lista."

### Qual é o problema

A frase não está factualmente errada, e é esse o risco: ela decora o rótulo e
perde tudo que o conceito explica. São três falhas.

**Primeira, trata como item de lista o que é um ponto de um eixo.** As quatro
categorias não são uma enumeração arbitrária. Elas respondem a uma única
pergunta, *quando a memória é alocada e liberada*, e se ordenam por quanto a
vinculação de armazenamento é adiada: estática, dinâmica da pilha, dinâmica do
heap explícita, dinâmica do heap implícita. Sem esse eixo, não há como decidir
entre elas.

**Segunda, ignora a relação com tempo de vida e escopo.** O que define a
variável dinâmica do heap não é o endereço, é o **desacoplamento** entre tempo
de vida e escopo. Numa variável da pilha os dois coincidem: ela nasce ao entrar
no bloco e morre ao sair. Numa do heap o objeto sobrevive ao escopo que o criou,
e é justamente isso que permite construir listas encadeadas, árvores e devolver
objetos de um método.

**Terceira, some com o custo.** Esse desacoplamento é a origem dos dois erros
clássicos de gerência de memória. O ponteiro pendente aparece quando a memória é
liberada e ainda existe referência para ela; o vazamento aparece quando não há
mais referência e a memória nunca é liberada. Nenhum dos dois é possível em
variáveis da pilha, e ambos definem a escolha entre desalocação manual, como o
`delete` do C++, e coleta de lixo, como em Java e Python.

### Conclusão reescrita

Variáveis dinâmicas do heap são alocadas e desalocadas em tempo de execução, por
comando explícito do programa, e seu diferencial não é a região de memória, e
sim o fato de o tempo de vida deixar de coincidir com o escopo. Esse
desacoplamento é o que viabiliza estruturas de tamanho variável e objetos que
sobrevivem ao método que os criou.

O mesmo desacoplamento cobra um preço: alguém precisa decidir quando liberar a
memória. Ou o programador decide, ganhando previsibilidade e correndo risco de
ponteiro pendente e vazamento, ou um coletor de lixo decide, ganhando segurança
e pagando com pausas e custo de execução. Escolher entre pilha e heap é escolher
entre tempo de vida amarrado ao escopo, mais barato e mais seguro, e tempo de
vida livre, mais caro e mais poderoso.

---

## 20. Escopo estático

> Explique o conceito para um colega usando uma definição, um exemplo autoral e
> uma relação com outro conceito da aula.

### Definição

Escopo estático, também chamado de léxico, significa que dá para descobrir a
qual declaração cada nome se refere **olhando apenas o texto do programa**, sem
executá-lo. Quando o compilador encontra um nome, ele procura no bloco onde o
nome aparece; se não achar, sobe para o pai estático, o bloco que **contém**
aquele no código-fonte, e repete até achar ou concluir que o nome não foi
declarado.

A palavra-chave é *contém*. O que vale é quem envolve quem no arquivo, e não
quem chamou quem durante a execução.

### Exemplo autoral

```python
limite = 100                  # (1) global

def validar(valor):
    limite = 50               # (2) local a validar
    def conferir():
        return valor <= limite   # qual limite?
    return conferir()

print(validar(80))            # False
```

A função `conferir` não declara `limite`. O compilador procura no corpo de
`conferir` e não encontra; sobe para o pai estático, que é `validar`, porque
`conferir` está escrita dentro dela, e encontra o `limite` da linha (2), que
vale 50. Como 80 não é menor ou igual a 50, o resultado é `False`.

O ponto decisivo: `conferir` se liga ao `limite` de `validar` porque está
**escrita** dentro dela, e não porque foi **chamada** por ela. Se a linguagem
usasse escopo dinâmico, a busca seguiria a cadeia de chamadas em vez da cadeia
de aninhamento, e o resultado poderia mudar conforme quem chamasse a função.

### Relação com outro conceito

O exemplo acima também é um caso de **ocultação de nomes**, o exercício 21. O
`limite` da linha (2) esconde o da linha (1) dentro de `validar`, e o global
fica inacessível por nome simples ali dentro. Ocultação é consequência direta da
regra de busca do escopo estático: como a procura para na primeira declaração
encontrada, a mais interna sempre vence.

A relação com **tempo de vida** é igualmente importante, e os dois conceitos são
independentes. O `limite` global tem escopo restrito ao módulo, mas tempo de
vida igual ao do programa inteiro. O `limite` de `validar` tem escopo menor
ainda e morre quando a função retorna. Escopo é *onde o nome é visível*, tempo
de vida é *quando a memória existe*, e confundir os dois é a origem de boa parte
dos erros com variáveis estáticas.

---

## Referências

- SEBESTA, R. W. *Conceitos de Linguagens de Programação*. 11. ed. Capítulo 5:
  Nomes, Vinculações e Escopo.
- Lista de exercícios autorais fornecida pelo professor ([exercicios.pdf](exercicios.pdf)).
