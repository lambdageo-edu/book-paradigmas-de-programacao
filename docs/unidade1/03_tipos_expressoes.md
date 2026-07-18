# Cap 3: Tipos e Expressões

## 🧮 Conceitos sobre tipos

### O que é um sistema de tipos

Um sistema de tipos é, essencialmente, um conjunto de regras para checar a consistência de um programa. Luca Cardelli resume bem o propósito: tipos existem para impor restrições que ajudam a garantir corretude — o objetivo fundamental de um sistema de tipos é prevenir a ocorrência de erros de execução.

Formalmente, um **tipo é um conjunto de valores** que compartilham um comportamento uniforme sob as operações associadas a esse conjunto:

* `v` é um valor do tipo `T` se `v ∈ T`.
* Exemplo: para `Boolean`, os valores são `{true, false}` e as operações são `and`, `or`, `not`. A cardinalidade do tipo, `#Boolean`, é 2.

### Checagem de tipos: estática × dinâmica

A checagem de tipos garante que uma operação só é aplicada a operandos compatíveis. Ela pode acontecer:

* **Em tempo de compilação (estática)**: toda variável e parâmetro tem um tipo conhecido antes da execução — Pascal, C, C++, Java, Kotlin, Swift, Haskell.
* **Em tempo de execução (dinâmica)**: apenas os *valores* carregam um tipo; variáveis podem assumir valores de tipos diferentes ao longo da execução — Python, Ruby, Lua, JavaScript.
* **Em ambos**: linguagens estaticamente tipadas quase sempre também fazem alguma checagem em tempo de execução (por exemplo, um `ClassCastException` em Java, ou divisão por zero).

Java é estaticamente tipada. O código abaixo nem compila, porque `max` espera dois `int` e recebemos `double`:

```java
public class App {
    public static int max(int a, int b) {
        return a > b ? a : b;
    }

    public static void main(String[] args) {
        double x = 1.5, y = 1.8;
        System.out.println(max(x, y)); // erro de compilação: incompatible types
    }
}
```

Em uma linguagem dinamicamente tipada, como Lua ou Python, a mesma função `max` aceitaria números ou strings sem reescrever nada — a checagem só aconteceria (ou não) quando o programa rodasse.

### Fortemente × fracamente tipada

Uma linguagem é **fortemente tipada** se erros de tipo são *sempre* detectados, seja em compilação ou execução — e **fracamente tipada** se permite conversões implícitas silenciosas que podem mascarar erros. Isso é ortogonal a estática/dinâmica: C é estaticamente tipada, mas fracamente (permite conversões implícitas perigosas); Python é dinamicamente tipada, mas fortemente (uma operação inválida sempre lança exceção, nunca produz lixo silencioso).

Java fica no meio do caminho: é razoavelmente forte, mas ainda permite algumas conversões implícitas entre tipos numéricos (*widening*) que podem surpreender:

```java
int total = 10 / 3;        // 3 — divisão inteira, sem aviso
double media = 10 / 3;     // 3.0 — o resultado int é convertido para double DEPOIS da divisão!
double correta = 10.0 / 3; // 3.333... — agora a divisão já é em ponto flutuante
```

Kotlin, por comparação, é mais estritamente tipada nesse ponto: `val x: Double = 10 / 5` é erro de compilação, porque `10 / 5` já é um `Int` e Kotlin não converte implicitamente `Int` para `Double` na atribuição.

### Vantagens e limites da tipagem estática

Cardelli resume as vantagens: tipagem estática permite detectar inconsistências de tipo em tempo de compilação, garante que o programa executado é internamente consistente quanto a tipos, favorece maior eficiência de execução (o compilador já sabe os tipos, não precisa checá-los em runtime) e impõe uma disciplina que tende a deixar o código mais estruturado e legível.

A contrapartida é a perda de flexibilidade: tipagem estática pode restringir prematuramente o comportamento de um valor a um único tipo, dificultando escrever, por exemplo, uma função de ordenação genérica que funcione uniformemente para qualquer tipo comparável. É exatamente esse problema que o **polimorfismo** resolve — veremos isso em detalhe na Unidade 2.

### Inferência de tipos

Linguagens estáticas modernas — Kotlin, Swift, Scala, Haskell e, parcialmente, o próprio Java — oferecem **inferência de tipos**: o compilador deduz o tipo de uma expressão sem que você precise anotá-lo explicitamente. Desde o **Java 10**, a palavra-chave `var` permite inferência de tipo para variáveis locais:

```java
var nome = "Sergio";     // o compilador infere String
var idade = 42;          // infere int
var notas = List.of(8.5, 9.0, 7.2); // infere List<Double>

// nome = 10; // erro! var NÃO torna a variável dinamicamente tipada —
              // o tipo é fixado em String na declaração, só a ANOTAÇÃO é opcional.
```

!!! warning "`var` não é tipagem dinâmica"
    Esse é o erro conceitual mais comum sobre `var`. O tipo continua sendo determinado em tempo de compilação — `var` só evita repetir o nome do tipo quando ele já é óbvio pelo lado direito da atribuição. Use com moderação: `var lista = new ArrayList<Pedido>()` é claro; `var resultado = processar(x)` pode obscurecer o tipo de retorno se `processar` não tiver um nome autoexplicativo.

### Classificação dos tipos de dados

Os tipos de dados se dividem em **primitivos** e **compostos**:

* **Primitivos**: valores atômicos, que não se decompõem em valores mais simples — `char`, `int`, `double`, `boolean`.
* **Compostos**: valores construídos a partir de valores mais simples — `struct`, array, união, registro, tupla, lista.

David Watt propõe uma categorização mais fina para tipos compostos:

* **Produtos cartesianos** (tuplas, structs, registros, classes)
* **Mapeamentos** (arrays, funções)
* **Uniões disjuntas** (tipos algébricos, registros discriminados, objetos)
* **Tipos recursivos** (listas, árvores)

## ☕ Tipos em Java

Java tem uma distinção fundamental entre dois grupos de tipos: **primitivos** e **de referência**.

### Tipos primitivos

Java define **8 tipos primitivos**, todos com tamanho fixo e portável (ao contrário de C, onde `int` pode variar de plataforma para plataforma):

| Tipo | Tamanho | Faixa / valores |
|---|---|---|
| `byte` | 8 bits | -128 a 127 |
| `short` | 16 bits | -32.768 a 32.767 |
| `int` | 32 bits | ≈ -2,1 a 2,1 bilhões |
| `long` | 64 bits | inteiro de precisão estendida |
| `float` | 32 bits | ponto flutuante de precisão simples |
| `double` | 64 bits | ponto flutuante de precisão dupla |
| `char` | 16 bits | um caractere UTF-16 |
| `boolean` | — | `true` ou `false` |

Valores primitivos **não são objetos** — não têm métodos, e vivem na *stack* (ou embutidos diretamente em um objeto), não no *heap*. Essa é uma diferença de design importante em relação a linguagens como Python ou Ruby, onde "tudo é objeto".

### Tipos de referência

Todo o resto — classes, interfaces, arrays, `String` — é um **tipo de referência**: a variável guarda uma referência para um objeto alocado no *heap*, não o objeto em si.

```java
int x = 10;              // x guarda o valor 10 diretamente
String s = "olá";        // s guarda uma referência para um objeto String no heap
int[] arr = {1, 2, 3};   // arr guarda uma referência para um array no heap
```

### Boxing, autoboxing e wrappers

Cada tipo primitivo tem uma classe *wrapper* correspondente (`int` → `Integer`, `double` → `Double`, ...), necessária sempre que um valor primitivo precisa ser tratado como objeto — por exemplo, dentro de uma coleção genérica, já que `List<int>` não é uma sintaxe válida:

```java
List<Integer> notas = new ArrayList<>();
notas.add(10);          // autoboxing: int 10 vira Integer automaticamente
int primeira = notas.get(0); // auto-unboxing: Integer vira int automaticamente
```

!!! warning "Cuidado ao comparar wrappers com `==`"
    `Integer a = 200; Integer b = 200; a == b` pode dar `false`, porque `==` em tipos de referência compara identidade de objeto, não valor — e o *cache* de `Integer` só garante identidade para valores entre -128 e 127. Use sempre `.equals()` (ou `Objects.equals`) para comparar wrappers por valor.

### Records: um novo tipo de produto cartesiano

Desde o **Java 16**, a linguagem tem *records* — uma forma concisa de declarar um tipo composto imutável (um produto cartesiano, na classificação de Watt), com construtor, *getters*, `equals`, `hashCode` e `toString` gerados automaticamente:

```java
public record Ponto(double x, double y) {
    // opcional: um construtor compacto para validação
    public Ponto {
        if (Double.isNaN(x) || Double.isNaN(y)) {
            throw new IllegalArgumentException("coordenadas inválidas");
        }
    }

    public double distancia(Ponto outro) {
        double dx = x - outro.x, dy = y - outro.y;
        return Math.sqrt(dx * dx + dy * dy);
    }
}
```

```java
var p1 = new Ponto(0, 0);
var p2 = new Ponto(3, 4);
System.out.println(p1.distancia(p2)); // 5.0
System.out.println(p1);               // Ponto[x=0.0, y=0.0] — toString automático
```

Compare com a versão "clássica" pré-Java 16, que exigia escrever manualmente construtor, *getters*, `equals`, `hashCode` e `toString` — dezenas de linhas de código repetitivo para o mesmo efeito. *Records* são o exemplo mais direto de como Java moderno absorveu ideias de linguagens funcionais (tipos de dados imutáveis, como em Haskell ou Kotlin) para reduzir código boilerplate.

### `sealed` — uniões disjuntas em Java

Desde o **Java 17**, classes e interfaces `sealed` permitem expressar diretamente uma **união disjunta** (na classificação de Watt) — um tipo que só pode ser um entre um conjunto fechado de subtipos conhecidos:

```java
public sealed interface Forma permits Circulo, Retangulo {}

public record Circulo(double raio) implements Forma {}
public record Retangulo(double largura, double altura) implements Forma {}
```

Combinado com *pattern matching* (próxima seção), isso permite ao compilador verificar exaustividade — algo que só linguagens com tipos algébricos, como Haskell ou Elixir, ofereciam tradicionalmente.

## ➕ Conceito sobre expressões

Uma **expressão** é uma construção que pode ser avaliada, produzindo um valor.

### Literais

O tipo mais simples de expressão é o literal — denota diretamente um valor fixo:

```java
365        // int
3.1416     // double
3.1416f    // float
false      // boolean
'%'        // char
"E aí?"    // String
```

!!! warning "Ponto flutuante e igualdade"
    Nunca compare `float`/`double` com `==` esperando igualdade exata — erros de arredondamento em ponto flutuante binário fazem `0.1 + 0.2 == 0.3` avaliar para `false` na maioria das linguagens, Java incluída. Para valores monetários ou onde precisão exata importa, use `BigDecimal`.

### Construtores

Um construtor é uma expressão que monta um valor composto a partir de suas partes. Em Java, isso é o operador `new`:

```java
var p = new Ponto(10, 20);       // Java (record)
val p = Point(10, 20)            // Kotlin, para comparação
p = Point 10 20                  -- Haskell, para comparação
```

### Chamadas de função e operadores

Uma chamada de função calcula um resultado aplicando uma função a argumentos: `f(x)`. Um **operador** é, conceitualmente, uma função associada a um símbolo e usada em notação infixa — `a + b` é equivalente a `+(a, b)`. Uma expressão como `a * b + c / d` é, no fundo, composição de funções: `+(*(a, b), /(c, d))`.

Java segue as regras usuais de **precedência** (multiplicação antes de adição) e **associatividade** (a maioria dos operadores binários é associativa à esquerda; exponenciação, quando existe, costuma ser à direita — embora Java, notavelmente, **não tenha operador de exponenciação**, usando `Math.pow(a, b)` no lugar).

### Expressões condicionais em Java

O **operador ternário** já existe desde sempre em Java e é a forma mais direta de uma expressão condicional:

```java
String status = nota >= 7.0 ? "aprovado" : "reprovado";
```

Mas até o **Java 13** (com *switch expressions* estabilizadas no Java 14), Java não tinha um equivalente direto ao `case`-como-expressão de Haskell ou ao `when` do Kotlin — só o `switch`-**comando** clássico, herdado de C, com *fall-through* implícito e sem valor de retorno. Isso mudou:

```java
// switch clássico (comando, não expressão) — ainda válido, mas verboso
int mes = 2;
int dias;
switch (mes) {
    case 2:
        dias = 28;
        break;
    case 4: case 6: case 9: case 11:
        dias = 30;
        break;
    default:
        dias = 31;
}
```

```java
// switch expression moderna (Java 14+) — concisa, sem fall-through, exaustiva
int dias = switch (mes) {
    case 2 -> 28;
    case 4, 6, 9, 11 -> 30;
    default -> 31;
};
```

Combinado com *records* e `sealed`, o Java 21 leva isso ainda mais longe com **pattern matching para `switch`** e **record patterns**, aproximando a linguagem do `case` de Haskell ou do `when` do Kotlin:

```java
sealed interface Forma permits Circulo, Retangulo {}
record Circulo(double raio) implements Forma {}
record Retangulo(double largura, double altura) implements Forma {}

static double area(Forma forma) {
    return switch (forma) {
        case Circulo c -> Math.PI * c.raio() * c.raio();
        case Retangulo(double l, double a) -> l * a; // record pattern: desestrutura direto
        // sem default: o compilador SABE que Forma só tem esses dois casos (sealed)
        // e recusa compilar se faltar algum
    };
}
```

Isso é uma mudança real de estilo: o compilador agora garante **exaustividade** — se alguém adicionar um terceiro tipo de `Forma` no futuro e esquecer de tratá-lo aqui, o código não compila. É o mesmo tipo de garantia que atrai tanta gente para tipos algébricos em linguagens funcionais.

### Text blocks (Java 15+)

Para strings literais multilinha — comum em SQL, JSON, HTML embutido — o Java 15 introduziu *text blocks*, delimitados por `"""`:

```java
String sql = """
    SELECT nome, email
    FROM contato
    WHERE ativo = true
    """;
```

Comparado à concatenação manual de strings com `+` e `\n`, que era a única opção até então, isso reduz bastante ruído visual em código que monta texto estruturado.

## 📦 Variáveis, amarração e escopo

Vale distinguir duas visões de "variável" entre paradigmas, como propõe David Watt:

* **Em linguagens funcionais/lógicas** (como em matemática), uma "variável" representa um valor fixo, porém desconhecido — não muda depois de definida.
* **Em linguagens imperativas/OO**, uma variável é um **contêiner** para um valor, que pode ser inspecionado e atualizado quantas vezes for preciso — modelando um objeto do mundo real cujo estado muda ao longo do tempo.

Java, por padrão, segue o modelo imperativo — variáveis são mutáveis a menos que marcadas `final`:

```java
int contador = 0;
contador = contador + 1;   // ok, contador é mutável

final int LIMITE = 100;
// LIMITE = 200;            // erro de compilação: LIMITE é final
```

**Amarração** (*binding*) é a associação entre um identificador e uma entidade — um valor, uma variável, uma função. Três perguntas organizam o estudo de amarrações em qualquer linguagem:

1. Que símbolos podem compor um identificador válido?
2. Que tipos de entidade podem ser amarrados a um identificador?
3. Qual é a validade dessa amarração no tempo e no espaço — isto é, seu **escopo**?

O **escopo** de uma declaração é a porção do programa onde ela produz efeito. Em linguagens antigas como Cobol, o escopo de toda declaração era o programa inteiro; nas linguagens modernas — Java incluída — o escopo é delimitado por **blocos** (`{ }`), o que permite reutilizar nomes de variáveis em blocos diferentes sem conflito.

## 📝 Atividades

1. Explique, em termos de checagem estática vs. dinâmica e forte vs. fraca, por que `10 / 3` e `10.0 / 3` produzem resultados diferentes em Java, mesmo compilando os dois sem erro.
2. Reescreva, usando `record`, uma classe `Pessoa` com `nome` e `idade` que você teria escrito "à moda antiga" (construtor, getters, `equals`, `hashCode`, `toString` manuais). Compare o tamanho dos dois códigos.
3. Modele um tipo `Resultado` com `sealed interface` que representa `Sucesso(String valor)` ou `Falha(String motivo)`, e escreva uma `switch expression` exaustiva sobre ele.
4. Pesquise a diferença entre `==` e `.equals()` para tipos de referência em Java, e explique por que `new String("a") == new String("a")` é `false`, mas `new String("a").equals(new String("a"))` é `true`.
5. Compare *switch expressions* de Java 21 com `when` em Kotlin e `case` em Haskell — que semelhanças e diferenças você identifica?

## 🔗 Para saber mais

* [JEP 395 — Records](https://openjdk.org/jeps/395)
* [JEP 409 — Sealed Classes](https://openjdk.org/jeps/409)
* [JEP 441 — Pattern Matching for switch](https://openjdk.org/jeps/441)
* Cardelli, L. — *On Understanding Types, Data Abstraction, and Polymorphism*
