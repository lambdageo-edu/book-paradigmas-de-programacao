# Cap 8: Classes Abstratas e Interfaces

## 🧩 Classes Abstratas

### O problema

Suponha que vamos implementar várias classes representando formas geométricas — `Circle`, `Rectangle`, `Triangle`. Independente do tipo, todas compartilham operações como `area()` e `circumference()`. É natural querer uma superclasse `Shape` que concentre esses métodos comuns — mas `Shape`, isoladamente, não representa nenhuma forma real, então não há como dar a `area()` uma implementação útil ali.

### Métodos e classes abstratas

Java resolve isso com **métodos abstratos**: um método declarado, mas sem corpo — apenas assinatura seguida de `;`:

```java
public abstract double area();
```

!!! note "Comparando com C++"
    Um método abstrato em Java equivale a uma função virtual pura em C++: `virtual double area() = 0;`. Métodos `static`, `private` e `final` não podem ser `abstract`, já que nenhum desses pode ser sobrescrito por uma subclasse — e "abstrato" só faz sentido para algo que uma subclasse vai preencher.

Regras que decorrem disso:

* Qualquer classe com pelo menos um método abstrato é, ela mesma, automaticamente abstrata, e precisa ser declarada `abstract`.
* Uma classe abstrata **não pode ser instanciada** diretamente.
* Uma subclasse só pode ser instanciada se implementar **todos** os métodos abstratos herdados — se não implementar, ela mesma continua abstrata.
* Uma classe `final` não pode ter métodos abstratos (não pode ter subclasses para implementá-los).
* Uma classe pode ser `abstract` mesmo sem nenhum método abstrato — um sinal de que a implementação está intencionalmente incompleta.

### Solução: `Shape` abstrata

```java
public abstract class Shape {
    public abstract double area();
    public abstract double circumference();
}
```

```java
class Circle extends Shape {
    public static final double PI = Math.PI;
    protected double r;

    public Circle(double r) { this.r = r; }

    public double getRadius() { return r; }

    @Override
    public double area() { return PI * r * r; }

    @Override
    public double circumference() { return 2 * PI * r; }
}

class Rectangle extends Shape {
    protected double w, h;

    public Rectangle(double w, double h) {
        this.w = w;
        this.h = h;
    }

    @Override
    public double area() { return w * h; }

    @Override
    public double circumference() { return 2 * (w + h); }
}
```

```java
List<Shape> shapes = List.of(
    new Circle(2.0),
    new Rectangle(1.0, 3.0),
    new Rectangle(4.0, 2.0)
);

double areaTotal = shapes.stream()
    .mapToDouble(Shape::area)
    .sum();
```

Nenhum *cast* é necessário para tratar `Circle` e `Rectangle` como `Shape` — e chamar `area()` sobre um `Shape` sempre executa a implementação correta via **ligação dinâmica** (Capítulo 7): a área de um círculo usa o código de `Circle`, a de um retângulo, o de `Rectangle`.

### Alternativa moderna: `sealed` + `record` para o mesmo problema

Vale conectar isso com o que vimos no Capítulo 3. Se as formas geométricas forem, por natureza, **imutáveis** (o caso comum), uma alternativa mais moderna a esse desenho com classe abstrata é modelar `Shape` como uma **interface `sealed`**, com cada forma concreta como um `record`:

```java
public sealed interface Shape permits Circle, Rectangle {}

public record Circle(double r) implements Shape {
    public double area() { return Math.PI * r * r; }
    public double circumference() { return 2 * Math.PI * r; }
}

public record Rectangle(double w, double h) implements Shape {
    public double area() { return w * h; }
    public double circumference() { return 2 * (w + h); }
}
```

A diferença prática: com `sealed`, o compilador conhece **exaustivamente** o conjunto de subtipos possíveis — um `switch` sobre `Shape` sem `default` compila se, e só se, cobrir `Circle` e `Rectangle`. Isso é útil quando o conjunto de formas é fixo e conhecido; a abordagem clássica com `abstract class` continua sendo a certa quando se espera que **novas subclasses sejam adicionadas livremente** por código externo (bibliotecas, plugins), já que `sealed` restringe justamente quem pode estender o tipo.

## 🔌 Interfaces

### Motivação

Às vezes é útil especificar **o que** uma classe deve fazer, sem se comprometer com **como**. Classes abstratas já cobrem parte disso — mas como Java só permite herança simples (Capítulo 6), uma classe só pode `extends` uma única superclasse. Para expressar múltiplos contratos independentes, Java oferece **interfaces**, que uma classe pode `implements` em qualquer quantidade.

Interfaces são comumente entendidas como **contratos**: ao implementar uma interface, uma classe se compromete a fornecer todo o comportamento que ela declara.

!!! note "Nomes diferentes, mesma ideia"
    O conceito de interface aparece em várias linguagens sob nomes diferentes — em Swift são *protocols*; em Rust, *traits*; em Go, *interfaces* implícitas (uma struct "implementa" uma interface automaticamente se tiver os métodos certos, sem declaração explícita). A ideia central — separar o contrato da implementação — é praticamente universal em linguagens estaticamente tipadas modernas.

### Definindo uma interface

```java
public interface Centered {
    void setCenter(double x, double y);
    double getCenterX();
    double getCenterY();
}
```

Algumas regras específicas de interfaces em Java:

* Por padrão, todo método de uma interface é implicitamente `abstract` e `public` — omitir esses modificadores é a convenção usual.
* Uma interface **não pode** ter variáveis de instância — apenas constantes `static final`. Interface é especificação pura, sem estado.
* Uma interface não pode ser instanciada e não tem construtor.
* Se uma classe implementa uma interface sem fornecer todos os métodos exigidos, ela mesma se torna `abstract`.
* Uma classe pode implementar **quantas interfaces quiser**.

```java
public class CenteredRectangle extends Rectangle implements Centered {
    private double cx, cy;

    public CenteredRectangle(double cx, double cy, double w, double h) {
        super(w, h);
        this.cx = cx;
        this.cy = cy;
    }

    @Override public void setCenter(double x, double y) { cx = x; cy = y; }
    @Override public double getCenterX() { return cx; }
    @Override public double getCenterY() { return cy; }
}
```

Interfaces são a espinha dorsal de várias bibliotecas centrais do Java — o Collections Framework é inteiramente organizado em torno de interfaces como `List`, `Set`, `Map` (veremos em detalhe na Unidade 3), e a API de JDBC é construída sobre interfaces como `Connection`, `Statement`, `ResultSet`, implementadas de forma diferente por cada fabricante de banco de dados.

### Interfaces desde o Java 8: métodos `default` e `static`

Antes do Java 8, interfaces só podiam declarar métodos abstratos — o que criava um problema real de evolução de API: adicionar um método novo a uma interface **quebrava** toda classe que já a implementava, já que ela precisaria escrever uma implementação nova. Isso não acontecia com classes abstratas, onde bastava adicionar um método concreto.

O Java 8 resolveu isso com **métodos `default`** — métodos de interface com corpo, que implementações existentes herdam automaticamente se não os sobrescreverem. Foi assim que o método `sort` pôde ser adicionado à interface `List` sem quebrar código legado:

```java
public interface Fool {
    void f1();
    int f2(int x);

    default void f3() {
        System.out.println("olá, sou um método default");
    }

    static void f4() {
        System.out.println("sou um método estático de interface");
    }
}
```

Desde o **Java 9**, interfaces também podem ter métodos **`private`** — úteis para compartilhar lógica interna entre métodos `default`, sem expor esse detalhe na API pública:

```java
public interface Validador {
    boolean valido(String entrada);

    default void validarOuFalhar(String entrada) {
        if (!valido(entrada)) {
            throw new IllegalArgumentException(mensagemErro(entrada));
        }
    }

    private String mensagemErro(String entrada) { // Java 9+: método privado de interface
        return "entrada inválida: " + entrada;
    }
}
```

### Interfaces funcionais

Uma **interface funcional** é uma interface com exatamente **um** método abstrato (métodos `default`/`static` não contam) — a base sobre a qual funcionam as expressões lambda do Java, que veremos em detalhe no capítulo de Java Funcional (Unidade 3):

```java
@FunctionalInterface
public interface Operacao {
    int aplicar(int a, int b);
}
```

```java
Operacao soma = (a, b) -> a + b; // lambda implementando Operacao.aplicar
System.out.println(soma.aplicar(2, 3)); // 5
```

## 🆚 Classe Abstrata × Interface: quando usar cada uma

| | Classe abstrata | Interface |
|---|---|---|
| Herança | Simples (`extends`, uma só) | Múltipla (`implements`, quantas quiser) |
| Estado (atributos de instância) | Pode ter | Não pode (só constantes `static final`) |
| Construtor | Pode ter | Não tem |
| Uso típico | "É um X, compartilhando implementação parcial" | "Sabe fazer X", contrato independente de hierarquia |

Uma regra prática comum: use **classe abstrata** quando várias classes relacionadas compartilham código de implementação além da assinatura; use **interface** quando você só precisa garantir um contrato, possivelmente entre classes que não têm relação hierárquica nenhuma entre si (por exemplo, `Comparable`, implementado por `String`, `Integer`, e qualquer classe sua).

## 📝 Atividades

1. Reescreva a hierarquia `Shape`/`Circle`/`Rectangle` do capítulo, adicionando uma terceira forma (`Triangle`) — primeiro com `abstract class`, depois com `sealed interface` + `record`. Em qual versão o compilador te avisa se você esquecer de tratar `Triangle` em um `switch`?
2. Explique, com suas palavras, por que a ausência de métodos `default` pré-Java 8 tornava a evolução de interfaces publicadas (como as do próprio JDK) um problema real de compatibilidade.
3. Modele uma interface `Pagavel` com um método abstrato `valorTotal()` e um método `default` `resumoPagamento()` que usa `valorTotal()`. Implemente-a em duas classes sem relação hierárquica entre si (por exemplo, `Pedido` e `Assinatura`).
4. Pesquise a interface `Comparable<T>` do Java e explique por que ela é um bom exemplo de "contrato independente de hierarquia de classes".

## 🔗 Para saber mais

* Flanagan, D. & Evans, B. — *Java in a Nutshell*, capítulo sobre classes abstratas e interfaces
* [JEP 213 — Milling Project Coin (métodos privados em interfaces)](https://openjdk.org/jeps/213)
