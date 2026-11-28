# Cap 15: Java Funcional

## 🌉 Trazendo ideias funcionais para uma linguagem imperativa

Retomando o Capítulo 2: Java não foi projetada para ser uma linguagem híbrida, e seu núcleo continua sendo orientado a objetos baseado em classes. Mas, principalmente a partir do **Java 8** (2014), a linguagem incorporou de forma consistente conceitos centrais do paradigma funcional:

* **Dados imutáveis** e isolamento de efeitos colaterais — não no sentido estrito de proibi-los (Java continua permitindo mutação livremente), mas de oferecer ferramentas para **isolar** onde eles acontecem.
* **Funções como valores de primeira classe** — o que possibilita funções de alta ordem como `map`, `filter` e `reduce`, no mesmo sentido amplo discutido no Capítulo 1.

## λ Expressões Lambda, Interfaces Funcionais e Method References

Antes do Java 8, "passar comportamento como argumento" exigia sempre uma **classe anônima** implementando uma interface com um único método — verboso mesmo para uma operação trivial:

```java
List<Integer> list = List.of(1, 2, 3, 4);

list.stream().reduce(new BinaryOperator<Integer>() {
    @Override
    public Integer apply(Integer t, Integer u) {
        return t * u;
    }
}).ifPresent(new Consumer<Integer>() {
    @Override
    public void accept(Integer t) {
        System.out.println(t);
    }
});
```

Com **notação lambda**, o mesmo código:

```java
list.stream().reduce((x, y) -> x * y).ifPresent(System.out::println);
```

Isso só é possível porque `BinaryOperator<T>` e `Consumer<T>` são **interfaces funcionais** — interfaces com exatamente um método abstrato, conceito já visto no Capítulo 8. Uma lambda é, tecnicamente, uma implementação anônima e concisa de uma interface funcional; o compilador infere qual interface implementar a partir do contexto de uso.

O `java.util.function` fornece as interfaces funcionais mais comuns, reutilizadas por toda a biblioteca padrão:

| Interface | Método abstrato | Uso típico |
|---|---|---|
| `Function<T, R>` | `R apply(T t)` | transformar um valor em outro |
| `Predicate<T>` | `boolean test(T t)` | testar uma condição |
| `Consumer<T>` | `void accept(T t)` | consumir um valor (efeito colateral) |
| `Supplier<T>` | `T get()` | fornecer um valor, sem argumentos |
| `BinaryOperator<T>` | `T apply(T a, T b)` | combinar dois valores do mesmo tipo |

Quando a lambda apenas chama um método já existente, uma **method reference** é ainda mais concisa:

```java
list.forEach(x -> System.out.println(x)); // lambda
list.forEach(System.out::println);        // method reference — mesmo efeito, mais direto
```

### Strategy com lambdas

O padrão **Strategy** (Capítulo 9) encapsula algoritmos intercambiáveis atrás de uma interface comum. Quando a estratégia é um único método, uma interface funcional e uma lambda bastam, sem criar uma classe para cada variação:

```java
static double precoFinal(double valor, DoubleUnaryOperator desconto) {
    return desconto.applyAsDouble(valor);
}
```

```java
DoubleUnaryOperator semDesconto = v -> v;
DoubleUnaryOperator estudante = v -> v * 0.5;

System.out.println(precoFinal(40.0, semDesconto)); // 40.0
System.out.println(precoFinal(40.0, estudante));   // 20.0
```

O próprio `sort` faz isso: o `Comparator` recebido é a estratégia de ordenação.

## 🌊 Java Stream API

A Stream API, também introduzida no Java 8, aplica esse repertório funcional ao processamento de **coleções** — reencontrando, de forma mais explícita, a comparação entre estilo imperativo e declarativo do Capítulo 1:

```java
List<String> nomes = List.of("Ana", "Bruno", "Carla", "Davi", "Elisa");

// imperativo
List<String> resultado = new ArrayList<>();
for (String nome : nomes) {
    if (nome.length() > 4) {
        resultado.add(nome.toUpperCase());
    }
}

// declarativo, com Stream
List<String> resultado2 = nomes.stream()
    .filter(nome -> nome.length() > 4)
    .map(String::toUpperCase)
    .toList(); // Java 16+; antes disso, era .collect(Collectors.toList())
```

Uma Stream é uma sequência de operações encadeadas, dividida em duas categorias:

* **Intermediárias** (`filter`, `map`, `sorted`, `distinct`...) — retornam outra Stream, permitindo encadeamento; são **preguiçosas** (*lazy*): nada é executado até que uma operação terminal seja chamada.
* **Terminais** (`collect`, `toList`, `forEach`, `reduce`, `count`, `sum`...) — disparam de fato o processamento e produzem um resultado (ou efeito).

```java
double media = List.of(8.5, 7.0, 9.2, 6.5).stream()
    .mapToDouble(Double::doubleValue)
    .average()
    .orElse(0.0);

Map<Boolean, List<String>> aprovados = List.of("Ana:8", "Bruno:5", "Carla:9").stream()
    .collect(Collectors.partitioningBy(s -> Double.parseDouble(s.split(":")[1]) >= 7));
```

!!! tip "Stream não é sempre a melhor escolha"
    Streams brilham em pipelines de transformação — filtrar, mapear, agregar. Para um laço simples com efeitos colaterais óbvios (por exemplo, atualizar um contador externo a cada iteração), um `for` tradicional continua sendo mais claro e, geralmente, tão eficiente quanto. Prefira Streams quando elas deixam a **intenção** mais legível — não como substituição automática de todo `for`.

## ❓ Optional: modelando "pode não haver valor"

Um problema recorrente em Java — e em muitas linguagens com referências nulas — é `NullPointerException`, já citado no Capítulo 1 como um dos fatores que reduzem confiabilidade. Desde o Java 8, `Optional<T>` oferece um jeito explícito de modelar "esse método pode não ter um resultado", sem depender de `null`:

```java
public Optional<Usuario> buscarPorId(String id) {
    Usuario u = repositorio.get(id); // pode ser null internamente
    return Optional.ofNullable(u);
}
```

```java
// sem Optional: o método devolveria null, e quem chama precisa se lembrar de checar
Usuario usuario = repositorio.get("u1");
if (usuario != null) { /* ... */ }

// o idiomático com Optional:
Optional<Usuario> resultado = buscarPorId("u1");

String nome = resultado
    .map(Usuario::getNome)
    .orElse("desconhecido");

resultado.ifPresentOrElse(
    u -> System.out.println("encontrado: " + u.getNome()),
    () -> System.out.println("usuário não encontrado")
);
```

!!! warning "`Optional` não é imposto pelo compilador"
    Diferente do sistema de tipos de Kotlin (`String?` vs. `String`) ou Swift (`String?`), `Optional` em Java é uma convenção de biblioteca, não uma garantia do compilador — nada impede que alguém retorne `null` em vez de `Optional.empty()`, ou chame `.get()` sem checar `.isPresent()` antes (o que lança `NoSuchElementException`, na prática recriando o mesmo problema que `Optional` tenta evitar). O ganho real de `Optional` é **de design de API**: o tipo de retorno já comunica, explicitamente, que o valor pode faltar — quem chama o método é lembrado disso pela própria assinatura, mesmo sem imposição do compilador.

## 🔀 Conectando com o capítulo de exceções: uma via alternativa a `try`/`catch`

Retomando o que ficou em aberto no Capítulo 10: `Optional` cobre bem o caso "valor pode não existir", mas não carrega informação sobre **por que** faltou. Para modelar falhas com uma explicação, sem lançar exceções, um padrão comum (embora não nativo do Java, diferente de Kotlin, com `Result`, ou Haskell, com `Either`) é definir um tipo `Resultado` próprio usando `sealed interface` e `record` (Capítulo 3):

```java
public sealed interface Resultado<T> permits Sucesso, Falha {}
public record Sucesso<T>(T valor) implements Resultado<T> {}
public record Falha<T>(String motivo) implements Resultado<T> {}
```

```java
static Resultado<Double> dividir(double a, double b) {
    if (b == 0) {
        return new Falha<>("divisão por zero");
    }
    return new Sucesso<>(a / b);
}
```

```java
Resultado<Double> r = dividir(10, 0);
String mensagem = switch (r) {
    case Sucesso<Double> s -> "resultado: " + s.valor();
    case Falha<Double> f -> "erro: " + f.motivo();
};
```

Isso não substitui as exceções, que continuam sendo a ferramenta certa para condições verdadeiramente excepcionais e não locais. O exemplo, porém, ilustra bem como os recursos vistos ao longo deste livro (interfaces `sealed`, *pattern matching*, generics) se combinam para trazer, de forma idiomática, uma técnica típica de linguagens funcionais para dentro de Java moderno.

## 📝 Atividades

1. Reescreva, usando Stream, um laço `for` que soma apenas os números pares de uma `List<Integer>`.
2. Implemente um método `Optional<String> primeiroMaiorQue(List<String> lista, int tamanho)` que retorna a primeira string com comprimento maior que `tamanho`, ou `Optional.empty()` se nenhuma existir.
3. Explique, com um exemplo, por que `Optional<T>` não deveria ser usado como tipo de um **atributo** de classe ou como tipo de **parâmetro** de método — apenas como tipo de **retorno** (essa é, inclusive, a recomendação oficial da equipe do Java).
4. Implemente o tipo `Resultado<T>` deste capítulo e use-o para modelar uma validação de cadastro que pode falhar por múltiplos motivos (nome vazio, e-mail inválido, idade negativa).

## 🔗 Para saber mais

* [Documentação oficial: Java Stream API](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/stream/Stream.html)
* [Documentação oficial: `Optional`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Optional.html)
* Pesquise sobre o tipo `Result` em Kotlin e o tipo `Either` em Haskell, para comparar com a modelagem de `Resultado<T>` proposta aqui.
