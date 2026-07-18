# Cap 6: Relações — Herança e Encapsulamento

Um programa orientado a objetos é uma coleção de objetos que colaboram trocando mensagens. Para modelar isso com clareza, a UML define três tipos de relacionamento entre classes, em ordem crescente de força.

## 🔗 Dependência ("usa um")

O relacionamento mais fraco: geralmente transiente, não se mantém durante toda a vida do objeto. Ocorre tipicamente dentro de um método, quando um objeto usa temporariamente um serviço de outro. Em um diagrama de classes, é representado por uma linha tracejada.

Em Java, dependência aparece como um parâmetro de método, uma variável local, ou o tipo de retorno de um método:

```java
public class RelatorioService {
    public void gerar(Pedido pedido) {   // depende de Pedido apenas durante a chamada
        var formatador = new FormatadorMoeda(); // depende de FormatadorMoeda localmente
        System.out.println(formatador.formatar(pedido.total()));
    }
}
```

## 🔗🔗 Associação ("tem um")

Um relacionamento estrutural mais forte que a dependência: indica que um objeto mantém uma referência a outro por um período estendido — tipicamente, como um atributo.

Associações têm três propriedades que valem a pena nomear explicitamente ao modelar um sistema:

* **Nome**: o que a relação significa (`Aluno` "matriculado em" `Turma`).
* **Navegabilidade**: de qual classe é possível "chegar" na outra. Uma seta explícita mostra a direção em que a navegação é possível.
* **Cardinalidade (multiplicidade)**: quantas instâncias de cada lado participam da relação — se omitida, assume-se 1.

Em Java, associação aparece como um **atributo** de instância:

```java
public class Window {
    private Rectangle bounds; // Window "tem um" Rectangle — associação
}
```

### Agregação

Lida como "tem um", mas os dois objetos têm ciclos de vida **independentes**. Uma `Window` tem um `Rectangle` que descreve sua posição e tamanho, mas o `Rectangle` poderia, em princípio, existir e ser usado por outra estrutura.

```java
public class Turma {
    private List<Professor> professores; // agregação: professores existem independentemente da turma
}
```

### Composição

Uma associação muito mais forte, ao ponto da **contenção**: os ciclos de vida das partes ficam presos ao ciclo de vida do todo. Se o objeto que possui a relação for destruído, as partes quase sempre são destruídas junto.

```java
public class Pedido {
    private List<ItemPedido> itens; // composição: um ItemPedido não existe sem seu Pedido
}
```

!!! tip "Um jeito rápido de distinguir agregação de composição"
    Pergunte: "essa parte faz sentido existir sozinha, sem o todo?" Um `ItemPedido` sem `Pedido` não faz sentido — composição. Um `Professor` sem uma `Turma` específica faz sentido perfeitamente (ele pode dar outras turmas, ou nenhuma) — agregação.

### Associações de 1-para-1 e 1-para-muitos

Em código, a cardinalidade "muitos" quase sempre se traduz em uma coleção (`List`, `Set`, `Map` — veremos essas estruturas em detalhe na Unidade 3):

```java
// 1 para 1
public class Pessoa {
    private CPF cpf;
}

// 1 para muitos
public class Turma {
    private List<Aluno> alunos;
}
```

## 🧬 Generalização ("é um") — Herança

Uma generalização é a relação entre um tipo mais geral (**superclasse**, ou pai) e um tipo mais específico (**subclasse**, ou filho) — também chamada de relação "é um": um `BayWindow` é um tipo de `Window`.

Uma classe pode ter zero, um ou mais pais. Uma classe sem pais mas com um ou mais filhos é uma **classe raiz**; uma classe sem filhos é uma **classe folha**. Uma classe com exatamente um pai usa **herança simples**; com mais de um, **herança múltipla**.

!!! warning "Java tem apenas herança simples de classes"
    Diferente de C++, que permite herança múltipla de classes, **Java permite que uma classe estenda apenas uma outra classe** (`extends`). É possível, porém, implementar **múltiplas interfaces** (`implements`) — veremos essa distinção com cuidado no próximo capítulo, e ela é uma das decisões de design mais citadas ao comparar Java com C++.

A herança modela naturalmente hierarquias de categorias (parecido com taxonomia biológica) e favorece **reusabilidade** — uma qualidade central no desenvolvimento de sistemas grandes. Retomando a ideia de TAD: capacidades de um tipo existente podem não atender totalmente às necessidades de uma aplicação cliente, exigindo adaptação — mas adaptar exigiria entender e mexer no código já existente. A herança oferece uma saída: **estender** um tipo sem modificar seu código original.

```java
public class Circle {
    private static final double PI = 3.14159265358979;
    private double r;

    public Circle(double r) { this.r = r; }
    public double circumference() { return 2 * PI * r; }
    public double area() { return PI * r * r; }
}
```

```java
public class PlaneCircle extends Circle {
    private double cx, cy;

    public PlaneCircle(double r, double x, double y) {
        super(r);           // chama o construtor da superclasse
        this.cx = x;
        this.cy = y;
    }

    public boolean isInside(double x, double y) {
        double dx = x - cx, dy = y - cy;
        return Math.sqrt(dx * dx + dy * dy) < getRadius();
    }
}
```

`PlaneCircle` reaproveita toda a lógica de `Circle` (área, circunferência) sem reescrevê-la, e adiciona um comportamento novo (`isInside`) que só faz sentido para um círculo posicionado em um plano.

!!! danger "Herança é uma ferramenta poderosa, mas fácil de abusar"
    Um erro comum de quem está aprendendo OO é usar herança para *qualquer* relação de reaproveitamento de código, mesmo quando a relação real não é "é um" — por exemplo, herdar `Pilha` de `ArrayList` só para reaproveitar `add`/`remove`, quando uma pilha não é, semanticamente, uma lista qualquer (ela restringe operações). A regra prática mais citada em projeto de software é *"prefira composição a herança"* quando a relação é "tem um" ou "usa um" comportamento de outra classe, reservando `extends` para quando a relação "é um" realmente se sustenta — veremos isso com mais profundidade no capítulo de Princípios e Padrões.

## 🧪 Laboratório: Rede Social

Modele um sistema com quatro classes principais — `SocialNetwork`, `User`, `Post` e `Comment` — usando as relações vistas neste capítulo: `SocialNetwork` é composta por uma coleção de `User`; cada `User` tem uma coleção de `Post`; cada `Post` tem uma coleção de `Comment`.

O sistema deve suportar o seguinte uso:

```java
public class SocialNetTest {
    public static void main(String[] args) {
        var rede = new SocialNetwork();

        rede.addUser(new User("u1", "Joao Carlos"));
        rede.addUser(new User("u2", "Maria Antonia"));
        rede.addUser(new User("u3", "Emanuel Benedito"));
        rede.addUser(new User("u4", "Josefina de Almeida"));

        rede.addFriendly("u1", "u2");
        rede.addFriendly("u1", "u3");
        rede.addFriendly("u2", "u4");

        rede.addPost("u1", new Post("p1", "Tomando café"));
        rede.addPost("u2", new Post("p2", "Amizade é tudo de bom"));
        rede.addPost("u2", new Post("p3", "Passeando com meu cachorro"));

        rede.addComment("u3", "u1", "p1",
                new Comment("c1", "Com pão de queijo? Tudo di bão!"));

        rede.liked("u4", "u2", "p2");

        rede.printUpdates(); // imprime posts por usuário, com seus comentários
    }
}
```

Ao implementar, identifique explicitamente: qual relação é composição (não sobrevive sem o todo) e qual é agregação (poderia, em teoria, ser reaproveitada em outro contexto).

## 📝 Atividades

1. Classifique cada par abaixo como dependência, associação (agregação ou composição) ou generalização, e justifique: `Motor`/`Carro`; `Pessoa`/`Endereco`; `Gerente`/`Funcionario`; `Metodo`/`Excecao` (uma exceção que um método pode lançar).
2. Por que Java não permite herança múltipla de classes? Pesquise ao menos um problema clássico que a herança múltipla de implementação pode causar (dica: "problema do diamante").
3. Reescreva o exemplo `Circle`/`PlaneCircle` usando composição em vez de herança (`PlaneCircle` **tem um** `Circle`, em vez de **ser um** `Circle`). Compare as duas versões: o que se ganha e o que se perde?

## 🔗 Para saber mais

* Booch, G. — *Object-Oriented Analysis and Design with Applications*
* [UML Class Diagrams — visão geral de relações](https://www.uml-diagrams.org/class-reference.html)
