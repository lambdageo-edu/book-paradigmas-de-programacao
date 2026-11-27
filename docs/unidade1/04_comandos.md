# Cap 4: Comandos

Um **comando** (*statement*) é uma construção executada por seu efeito — tipicamente, atualizar variáveis ou produzir efeitos colaterais (E/S, mutação de estado). Comandos são a unidade fundamental das linguagens **imperativas**; em linguagens puramente funcionais, a noção equivalente é a de *expressão* (capítulo anterior), avaliada por seu valor, não por seu efeito.

## 🚫 Skip

O comando *skip* não tem efeito algum. Em Java, é simplesmente `;` sozinho (um comando vazio) — raro na prática, mas legal:

```java
if (condicao) ; // comando vazio — geralmente sinal de bug, não de intenção
```

Comparar com `pass` em Python ou `null;` em Ada, que cumprem o mesmo papel de forma mais explícita e menos propensa a erro por esquecimento.

## 🟰 Atribuição

O comando mais comum em linguagens imperativas: armazena o valor de uma expressão `E` em uma variável `V`.

```java
int v = 10;              // atribuição simples
int v1, v2, v3;
v1 = v2 = v3 = 10;        // atribuição múltipla — associa à direita
int a = 5;
a += 10;                  // atribuição composta: equivale a  a = a + 10;
```

Java suporta atribuição composta para todos os operadores aritméticos e bit a bit: `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=`, `>>>=`.

Java **não** tem atribuição de tuplas nativa como Python (`a, b, c = 1, 2, 3`) — o mais próximo é desestruturar um *record* via *pattern matching* (visto no capítulo anterior), que é conceitualmente diferente: não atribui a variáveis existentes, mas declara novas.

!!! note "Pattern matching não é atribuição"
    Em linguagens funcionais como Elixir, o operador `=` não é atribuição — é **casamento de padrões**. `x = 1` associa `1` ao símbolo `x`; mas `1 = x` também é válido, pois o interpretador consegue casar os dois lados. Java não tem esse mecanismo para variáveis comuns, mas os *record patterns* do Java 21 (`case Retangulo(var l, var a) -> ...`, visto no capítulo anterior) são um parente distante dessa ideia — aplicado apenas em contextos de `switch`/`instanceof`, não como comando geral.

## 📞 Chamadas de procedimento (métodos `void`)

Um procedimento é uma coleção de comandos parametrizável, que existe primariamente por seus efeitos colaterais — pode modificar parâmetros, variáveis globais/de instância, ou realizar E/S. Em Java, o equivalente é um **método com retorno `void`**:

```java
static void imprimirRelatorio(List<Pedido> pedidos) {
    for (Pedido p : pedidos) {
        System.out.println(p.descricao());   // efeito colateral: E/S
    }
}
```

!!! tip "Procedimentos vs. funções"
    Funções são semanticamente modeladas como funções matemáticas — idealmente sem efeitos colaterais, sempre devolvendo um valor determinado apenas pelos argumentos. Em Java, "função" no sentido estrito corresponde a um método `static` **puro**: sem efeitos colaterais, sem depender de estado mutável externo. Vale ter esse vocabulário em mente ao projetar APIs — métodos que só calculam algo devem, na medida do possível, ser puros; métodos que produzem efeitos devem deixar isso claro (por nome, por assinatura `void`, ou por convenção como o sufixo `!` em outras linguagens).

Em Java, a passagem de parâmetros é **sempre por valor** — inclusive para objetos, onde o "valor" passado é a *referência* ao objeto (não o objeto em si). Isso significa que reatribuir o parâmetro dentro do método não afeta a variável do chamador, mas mutar o objeto apontado, sim:

```java
static void naoMudaReferencia(int[] arr) {
    arr = new int[]{99, 99}; // só reatribui a variável local arr — não afeta o chamador
}

static void mudaConteudo(int[] arr) {
    arr[0] = 99;              // muta o array apontado — o chamador VÊ essa mudança
}
```

## ➡️ Comandos sequenciais

A construção de controle de fluxo mais fundamental é a **sequência**: o comando `C2` é executado após `C1`. Em Java (como em C), a sequência dentro de um bloco é indicada por `;` entre comandos e o bloco é delimitado por `{ }`:

```java
{
    int a = 2;
    int b = 4;
    int c = a * b;
}
```

## 🔀 Comandos de seleção

Permitem escolher entre dois ou mais caminhos de execução.

### `if` / `else`

A forma introduzida pelo Algol60 e usada, com pequenas variações sintáticas, por praticamente toda linguagem imperativa desde então:

```java
if (nota >= 7.0) {
    aprovado = true;
} else {
    aprovado = false;
}
```

A cláusula `else` é opcional; múltiplos caminhos se expressam encadeando `else if`:

```java
if (nota >= 9.0) {
    conceito = "A";
} else if (nota >= 7.0) {
    conceito = "B";
} else if (nota >= 5.0) {
    conceito = "C";
} else {
    conceito = "D";
}
```

### `switch`

Java herda o `switch` de C — um comando de seleção múltipla que compara uma expressão contra um conjunto de constantes:

```java
switch (mes) {
    case 12: case 1: case 2:
        estacao = "verão";
        break;
    case 3: case 4: case 5:
        estacao = "outono";
        break;
    default:
        estacao = "desconhecida";
}
```

!!! danger "A armadilha clássica do `switch`: fall-through"
    Esquecer o `break` é um dos bugs mais famosos de C/Java: sem ele, a execução "cai" (*falls through*) para o próximo `case`, executando-o também. É por isso que o Java moderno (14+) introduziu a **switch expression** com `->`, vista no capítulo anterior — ela elimina o *fall-through* por padrão e ainda devolve um valor, unificando comando e expressão. Em código novo, prefira sempre a forma com `->` ao `switch` clássico com `:`/`break`.

Desde o Java 21, o `switch` também aceita **pattern matching sobre tipos**, não só constantes — permitindo testar e desestruturar o tipo de um objeto diretamente no `case` (veja o exemplo com `sealed interface Forma` no capítulo anterior).

## 🔁 Comandos iterativos

Sem iteração, seria preciso escrever cada ação repetida explicitamente em sequência — inviável na prática. Um comando iterativo tem tipicamente uma **condição** e um **corpo**, executado repetidamente enquanto a condição for verdadeira.

### `while`

A forma mais fundamental, também introduzida pelo Algol: avalia a condição; se verdadeira, executa o corpo e repete; senão, termina.

```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    i++;
}
```

Com apenas `while` (mais atribuição e seleção) já é possível expressar qualquer algoritmo computável — as demais formas de laço existem por conveniência de leitura/escrita, não por necessidade teórica.

### `do`-`while`

Variação em que a condição é testada **depois** do corpo — garante que o corpo execute ao menos uma vez:

```java
int tentativa;
do {
    tentativa = lerEntradaUsuario();
} while (tentativa < 0);
```

### `for`

Java segue a notação flexível de C, com três expressões separadas por `;`: inicialização, condição de parada, incremento.

```java
for (int i = 0; i < 10; i++) {
    System.out.println(i);
}
```

É, por definição, açúcar sintático para um `while` equivalente:

```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    i++;
}
```

### `for-each` (desde o Java 5)

Para percorrer coleções e arrays, Java oferece uma forma mais legível que dispensa índice explícito — o equivalente ao `for x in xs` de Python ou ao `for x <- xs` de Elixir:

```java
List<String> nomes = List.of("Ana", "Bruno", "Carla");
for (String nome : nomes) {
    System.out.println(nome);
}
```

Por baixo dos panos, o `for-each` usa a interface `Iterable`/`Iterator` — veremos isso com mais detalhe no capítulo de Coleções (Unidade 3), onde `Iterator` aparece também como exemplo de padrão de projeto.

### Um quarto caminho: laços como expressão funcional

Vale antecipar algo que retomaremos na Unidade 3: desde o Java 8, muito código que antes seria escrito como um `for` explícito hoje é escrito de forma **declarativa**, via Stream API — mais parecido com `map`/`filter` de linguagens funcionais do que com um laço imperativo clássico:

```java
// imperativo — "como" fazer, passo a passo
List<String> comFor = new ArrayList<>();
for (String nome : nomes) {
    if (nome.length() > 3) {
        comFor.add(nome.toUpperCase());
    }
}

// declarativo — "o que" queremos, via Stream
List<String> comStream = nomes.stream()
    .filter(nome -> nome.length() > 3)
    .map(String::toUpperCase)
    .toList();
```

Os dois produzem o mesmo resultado — mas ilustram bem a diferença entre paradigma imperativo e declarativo discutida no Capítulo 1, dentro da mesma linguagem.

## 📝 Atividades

1. Reescreva um laço `for` clássico que soma os números de 1 a 100 (a) como `while`, (b) como Stream com `IntStream.rangeClosed(1, 100).sum()`. Compare legibilidade.
2. Explique, com um exemplo concreto, por que esquecer o `break` em um `switch` clássico pode causar um bug silencioso — e mostre a mesma lógica reescrita como *switch expression* (`->`), livre desse risco.
3. Escreva um método `void` que recebe um `List<Integer>` e dobra cada elemento *in place* — depois explique, em termos de passagem por valor, por que isso funciona mesmo a lista sendo "passada por valor".
4. Pesquise a construção `for` do Python (`for x in range(...)`) e compare com o `for-each` de Java: que semelhanças/diferenças de design você percebe?

## 🔗 Para saber mais

* Watt, D. — *Programming Language Design Concepts*, capítulo sobre comandos e controle de fluxo
* [JEP 361 — Switch Expressions](https://openjdk.org/jeps/361)
