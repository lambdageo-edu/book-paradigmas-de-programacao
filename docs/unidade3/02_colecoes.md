# Cap 11: Coleções

## 📦 O que é o Java Collections Framework

Segundo a documentação da Oracle, uma **coleção** é um objeto que representa um grupo de objetos, e o *Collections Framework* é uma arquitetura unificada para representar e manipular coleções, permitindo trabalhar com elas de forma independente dos detalhes de implementação — a mesma ideia de interface como contrato que vimos no Capítulo 8, aplicada em larga escala pela própria biblioteca padrão do Java.

### Evolução

| Versão | Novidade |
|---|---|
| Java 5 | Generics aplicados às coleções — antes, `Vector` e companhia guardavam apenas `Object`, exigindo *cast* manual em todo acesso |
| Java 6 | `Deque` — fila de duas pontas |
| Java 8 | Stream API — processamento declarativo sobre coleções (veremos em detalhe no capítulo de Java Funcional) |
| Java 9 | Métodos de fábrica imutáveis — `List.of(...)`, `Set.of(...)`, `Map.of(...)` |
| Java 10 | `List.copyOf`, `toUnmodifiableList()` em Streams |

### Hierarquia de interfaces

O Collections Framework é organizado, deliberadamente, em torno de **interfaces** — não de classes concretas. Isso significa que código cliente deveria, na prática, quase sempre declarar variáveis pelo tipo de interface (`List`, `Set`, `Map`), não pela implementação concreta (`ArrayList`, `HashSet`), justamente para poder trocar a implementação depois sem afetar o restante do código — uma aplicação direta do princípio de "programe para interfaces, não para implementações" (Capítulo 9).

```
Iterable
  └── Collection
        ├── List     (ArrayList, LinkedList, Vector...)
        ├── Set      (HashSet, LinkedHashSet, TreeSet...)
        └── Queue    (LinkedList, PriorityQueue...)
              └── Deque (ArrayDeque, LinkedList...)

Map (não é Collection!)
        (HashMap, LinkedHashMap, TreeMap...)
```

!!! note "`Map` não é um `Collection`"
    Uma pegadinha comum: `Map<K, V>` **não** estende `Collection` — semanticamente, um mapa não é "uma coleção de elementos", é uma associação chave-valor. Para percorrer um `Map` como coleção, é preciso pedir explicitamente uma *view*: `map.keySet()`, `map.values()` ou `map.entrySet()`.

## 📋 List

Uma sequência ordenada, com acesso por índice, que permite elementos duplicados. As duas implementações mais usadas têm características de desempenho bem diferentes:

```java
List<String> nomes = new ArrayList<>(); // array redimensionável — acesso O(1) por índice
nomes.add("Ana");
nomes.add("Bruno");
nomes.add(0, "Zeca"); // inserir no início: O(n), precisa deslocar tudo

List<String> fila = new LinkedList<>(); // lista duplamente encadeada — inserção nas pontas O(1)
```

Para listas **imutáveis** — comuns quando você quer expor dados sem risco de o cliente alterá-los por engano — desde o Java 9:

```java
List<String> constantes = List.of("A", "B", "C");
// constantes.add("D"); // lança UnsupportedOperationException em tempo de execução
```

## 🔁 Queue e Deque

`Queue` modela uma fila **FIFO** (primeiro a entrar, primeiro a sair):

```java
Queue<String> fila = new LinkedList<>();
fila.add("Joao");
fila.add("Maria");
fila.add("Jose");

for (String nome : fila) {
    System.out.println(nome); // percorre sem remover: Joao, Maria, Jose
}

System.out.println(fila.peek()); // Joao — olha o próximo, sem remover
System.out.println(fila.poll()); // Joao — remove e retorna o próximo
System.out.println(fila.poll()); // Maria
```

`Deque` (*double-ended queue*) generaliza isso permitindo inserção/remoção nas duas pontas — e, por isso, também serve como implementação de **pilha** (LIFO):

```java
Deque<String> pilha = new ArrayDeque<>(); // preferível a LinkedList para uso como pilha hoje

pilha.push("Joao");
pilha.push("Maria");
pilha.push("Jose");

System.out.println(pilha.peek()); // Jose — o último empilhado
System.out.println(pilha.pop());  // Jose — remove o topo
System.out.println(pilha.pop());  // Maria
```

!!! tip "`Vector` e `Stack` são história, não recomendação"
    O material didático mais antigo de Java frequentemente ensina `Stack` (que estende `Vector`) para pilhas. Ambas as classes são de antes do Collections Framework (pré-Java 1.2) e têm sobrecarga desnecessária de sincronização para uso single-thread. A própria documentação da Oracle recomenda `ArrayDeque` no lugar de `Stack`, e `ArrayList` no lugar de `Vector`, para código novo.

## 🔢 Set

Uma coleção **sem elementos duplicados**, modelando o conceito matemático de conjunto:

```java
Set<String> visitantes = new HashSet<>();    // sem ordem garantida, O(1) amortizado
Set<String> ordenados = new TreeSet<>();     // mantém ordem natural (ou um Comparator)
Set<String> insercao = new LinkedHashSet<>(); // preserva a ordem de inserção
```

## 🗺️ Map

Uma associação chave→valor, sem chaves duplicadas:

```java
Map<String, Double> precos = new HashMap<>();
precos.put("café", 12.5);
precos.put("chá", 9.0);

System.out.println(precos.get("café")); // 12.5
System.out.println(precos.getOrDefault("suco", 0.0)); // 0.0 — evita NullPointerException

Map<String, Double> imutavel = Map.of("café", 12.5, "chá", 9.0); // Java 9+
```

## ⚖️ `equals` e `hashCode`

Um detalhe fácil de ignorar, mas que quebra silenciosamente `HashSet` e `HashMap` se feito errado: essas estruturas dependem de `hashCode()` para localizar rapidamente um elemento, e de `equals()` para confirmar a igualdade dentro do mesmo "balde" de hash. **O contrato entre os dois é obrigatório**: se `a.equals(b)` é `true`, então `a.hashCode() == b.hashCode()` também precisa ser. Sobrescrever um sem o outro é uma fonte clássica de bugs difíceis de rastrear — um objeto "desaparece" de um `HashSet` porque seu `hashCode` não bate mais com o que foi usado para inseri-lo.

```java
public class Ponto {
    private final int x, y;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Ponto p)) return false; // pattern matching de instanceof (Java 16+)
        return x == p.x && y == p.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
}
```

Como vimos no Capítulo 3, um `record` gera `equals`/`hashCode` corretos automaticamente, com base em todos os componentes — é uma das razões práticas para preferi-lo sempre que a classe representa apenas dados imutáveis.

## 🔄 Iterator: o padrão de projeto por trás do `for-each`

O `for-each`, visto no Capítulo 4, funciona para **qualquer** classe que implemente a interface `Iterable<T>` — e é exatamente esse mecanismo que possibilita percorrer `List`, `Set` e até coleções definidas por você, de forma uniforme:

```java
public interface Iterable<T> {
    Iterator<T> iterator();
}

public interface Iterator<T> {
    boolean hasNext();
    T next();
}
```

Esse par de interfaces é, literalmente, o padrão de projeto **Iterator** (Capítulo 9) implementado como parte central da linguagem: ele permite acessar sequencialmente os elementos de uma coleção **sem expor sua representação interna** — o código cliente não precisa saber se por trás existe um array, uma lista encadeada, ou uma árvore.

```java
List<String> nomes = List.of("Ana", "Bruno", "Carla");
Iterator<String> it = nomes.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}

// o for-each é, por baixo, exatamente este código:
for (String nome : nomes) {
    System.out.println(nome);
}
```

## 📝 Atividades

1. Implemente uma classe `Fila<T>` própria, usando `Deque<T>` internamente, expondo apenas os métodos que fazem sentido para uma fila (`enfileirar`, `desenfileirar`, `estaVazia`) — esconda a interface mais ampla do `Deque` por trás de uma API mais restrita (relacione com o Princípio de Segregação de Interfaces do Capítulo 9).
2. Explique por que `ArrayList` tem inserção O(1) no fim mas O(n) no início, enquanto `LinkedList` é O(1) em ambas as pontas — e por que, mesmo assim, `ArrayList` costuma ser a escolha padrão recomendada para a maioria dos casos de uso.
3. Escreva uma classe `Pessoa` com `nome` e `cpf`, sobrescrevendo `equals`/`hashCode` com base apenas no `cpf`. Insira duas instâncias com o mesmo CPF (mas nomes diferentes) em um `HashSet<Pessoa>` e explique o resultado.
4. Implemente `Iterable<Integer>` em uma classe `IntervaloInclusivo` que gera os números de `a` até `b`, permitindo usá-la diretamente em um `for-each`.

## 🔗 Para saber mais

* [Documentação oficial: Java Collections Framework Overview](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)
* [Java Collections Framework — Effective Java, capítulos sobre `equals`/`hashCode`](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
