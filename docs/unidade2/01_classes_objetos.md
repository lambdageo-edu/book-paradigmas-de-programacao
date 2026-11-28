# Cap 5: Classes e Objetos

Com os fundamentos de sintaxe cobertos na Unidade 1, entramos agora no núcleo da disciplina: **orientação a objetos**, usando Java como estudo de caso.

## 📜 Um pouco de história

Em 1989, na conferência OOPSLA (*Object-Oriented Programming, Systems, Languages & Applications*), pesquisadores tentaram sistematizar o que é, de fato, programação orientada a objetos — de onde vem, e quais objetivos persegue. Essa discussão foi organizada por Peter Wegner em torno de três conceitos centrais:

* Objetos são coleções de operações que compartilham um estado.
* Classes servem como *templates* a partir dos quais objetos são criados.
* Herança permite reaproveitar o comportamento de uma classe na definição de novas classes.

Alan Kay, criador do Smalltalk — uma das linguagens fundadoras do paradigma — resume o modelo mental em seis pontos: tudo é um objeto; objetos se comunicam trocando mensagens; cada objeto tem sua própria memória; todo objeto é instância de uma classe; a classe concentra o comportamento compartilhado pelas suas instâncias; e a execução de um programa é, no fundo, o primeiro objeto recebendo uma mensagem que se propaga.

Luca Cardelli, em 1984, propôs uma comparação esclarecedora com o paradigma estruturado: enquanto linguagens como Pascal organizam dados como produtos cartesianos (registros), somas disjuntas (uniões) e espaços de função (procedimentos) — uma visão que vem diretamente da matemática — a orientação a objetos organiza dados como uma **hierarquia de classes e subclasses**, onde dados em qualquer nível herdam os atributos dos níveis acima — uma visão que se aproxima mais da biologia e da taxonomia.

## 🧩 Conceitos principais

### Objeto

Um objeto é frequentemente descrito, de forma simplificada, como "dados com métodos" — mas essa definição é limitada e pouco útil na hora de modelar um sistema. Uma perspectiva mais produtiva é pensar em um objeto como algo que tem uma **responsabilidade**, concretizada através de suas ações (métodos). Um objeto é único e identificável, e precisa manter informação sobre seu estado atual através de seus **atributos** (também chamados de variáveis de instância).

Em resumo, um objeto requer três coisas:

* **Estado**: implementado por um conjunto de atributos.
* **Comportamento**: como o objeto reage a requisições de outros objetos.
* **Identidade única**: o que distingue esse objeto de todos os demais, mesmo que tenham o mesmo estado.

Pensar em objetos dessa forma — como responsabilidades, não como estruturas de dados — tende a melhorar duas propriedades centrais de qualquer componente de software: **coesão** (um objeto com poucas responsabilidades tende a ter operações fortemente relacionadas entre si) e **baixo acoplamento** (cada objeto cuida da sua própria responsabilidade, sem depender demais de detalhes internos de outros).

### Mensagens

Para que um objeto cumpra sua responsabilidade, é preciso que exista uma interface pública que permita a comunicação com outros objetos — feita por **troca de mensagens**, que a nível de implementação corresponde a chamadas de métodos públicos. Para enviar uma mensagem, é necessário: identificar o objeto receptor, identificar o método a executar, e passar os argumentos exigidos por esse método.

```java
conta.depositar(100.0);
//  ^objeto     ^mensagem (método + argumento)
```

### Classes

Um objeto é único, mas na prática lidamos com **conjuntos** de objetos que compartilham características — em um sistema acadêmico, todos os `Aluno` compartilham a mesma estrutura de dados e comportamento, ainda que cada um tenha seu próprio estado. Essas características comuns são agrupadas em uma **classe**, que define:

* Os elementos de dado que cada objeto contém (atributos).
* Os métodos que cada objeto pode executar.
* Como esses elementos podem ser acessados de fora da classe.

David Watt define classe como uma família de objetos com componentes variáveis e métodos similares. Uma analogia comum é pensar em classes como **fábricas de objetos**: em Java, o operador `new` constrói um novo objeto a partir de uma classe, e dizemos que cada objeto resultante é uma **instância** dessa classe.

```java
public class Aluno {
    private String nome;
    private double media;

    public Aluno(String nome, double media) {
        this.nome = nome;
        this.media = media;
    }

    public boolean isAprovado() {
        return media >= 7.0;
    }

    public String getNome() {
        return nome;
    }
}
```

```java
Aluno a1 = new Aluno("Ana", 8.5);
Aluno a2 = new Aluno("Bruno", 5.2);
System.out.println(a1.getNome() + ": " + a1.isAprovado()); // Ana: true
```

### Classes e tipos abstratos de dados

Vale conectar isso com o que já vimos no Capítulo 1: orientação a objetos pode ser entendida, em boa medida, como **tipos abstratos de dados + herança + polimorfismo**. Uma classe, sem herança nem polimorfismo, já é essencialmente um TAD: um conjunto de valores (os possíveis estados de um objeto) equipado com operações (os métodos), onde a implementação interna fica escondida atrás de uma interface pública. Compreender TADs é o que separa uma classe "de verdade" — coesa, com uma responsabilidade clara — de uma classe que é só um "carregador de dados soltos", pouco mais que um `struct` com métodos anexados.

!!! note "Nem toda orientação a objetos é baseada em classes"
    A maioria das linguagens orientadas a objetos populares — C++, Java, Kotlin, Swift — é *class-based*. Mas o conceito de classe não é estritamente necessário: linguagens como Lua e o JavaScript pré-ES6 (2015) implementam orientação a objetos via **protótipos**, onde novos objetos são criados clonando e ajustando um objeto-modelo, em vez de instanciados a partir de uma classe. O JavaScript moderno tem sintaxe `class`, mas ela ainda é, por baixo, açúcar sintático sobre protótipos.

    ```javascript
    function Ponto(a, b) {
        this.x = a;
        this.y = b;
    }
    Ponto.prototype.dist = function (outro) {
        return Math.hypot(this.x - outro.x, this.y - outro.y);
    };

    const p1 = new Ponto(10, 20);
    const p2 = new Ponto(30, 20);
    console.log(p1.dist(p2));
    ```

### Encapsulamento

O conceito de encapsulamento já existia nos tipos abstratos de dados, mas foram as linguagens orientadas a objetos que o tornaram um mecanismo de primeira classe da linguagem. Encapsulamento é a técnica de **minimizar as interdependências entre módulos**, escondendo atributos e detalhes de implementação e expondo apenas uma interface controlada.

As vantagens práticas são diretas: se os detalhes internos de uma classe estão escondidos, é possível modificar essa implementação livremente sem quebrar código que já depende dela — desde que a interface pública permaneça estável. Além disso, quando todos os dados são acessados exclusivamente através de métodos, esses métodos definem as únicas operações possíveis sobre o objeto; uma vez testados, ganha-se confiança de que o objeto sempre estará em um estado consistente. Se os atributos pudessem ser manipulados diretamente de fora, o número de combinações possíveis de estado se tornaria rapidamente impossível de testar.

Em Java, encapsulamento é implementado via **modificadores de acesso**:

```java
public class ContaBancaria {
    private double saldo;       // não acessível fora da classe

    public ContaBancaria(double saldoInicial) {
        this.saldo = saldoInicial;
    }

    public void depositar(double valor) {
        if (valor <= 0) {
            throw new IllegalArgumentException("valor deve ser positivo");
        }
        saldo += valor;
    }

    public double getSaldo() {
        return saldo;   // acesso somente leitura ao estado interno
    }
}
```

Sem o `private`, qualquer código externo poderia fazer `conta.saldo = -1000`, contornando toda validação — é exatamente esse tipo de "estupidez acidental ou proposital" (para usar a expressão de *Java in a Nutshell*) que o encapsulamento existe para prevenir.

Java tem quatro níveis de acesso, do mais restrito ao mais aberto:

| Modificador | Quem acessa |
|---|---|
| `private` | apenas a própria classe |
| nenhum (*package-private*) | classes do mesmo pacote |
| `protected` | classes do mesmo pacote e subclasses, mesmo em outros pacotes |
| `public` | qualquer classe |

### Membros de instância × membros estáticos

Um **atributo de instância** tem uma cópia em cada objeto. Um atributo **estático** (`static`) pertence à classe: há uma única cópia, compartilhada por todas as instâncias. Do mesmo modo, um **método de instância** é chamado sobre um objeto e enxerga o estado dele (`this`), enquanto um **método estático** é chamado pela classe e não tem `this`. Já uma **variável local** é declarada dentro de um método e só existe durante a execução daquele bloco.

```java
public class Matricula {
    private static int total = 0;   // um só, para a classe toda
    private final int numero;       // um por objeto

    public Matricula() {
        total++;
        int proximo = total;        // variável local do construtor
        this.numero = proximo;
    }

    public int getNumero() { return numero; }      // método de instância
    public static int getTotal() { return total; } // método estático
}
```

```java
var m1 = new Matricula();
var m2 = new Matricula();
System.out.println(m1.getNumero() + " " + m2.getNumero()); // 1 2
System.out.println(Matricula.getTotal());                  // 2
```

### Records como objetos de estado imutável

Vale revisitar aqui o *record*, introduzido no Capítulo 3: para objetos cujo papel é apenas **carregar um estado imutável** (sem comportamento complexo nem necessidade de encapsulamento mutável), *record* é hoje a forma idiomática em Java — evita boilerplate sem abrir mão de encapsulamento (os campos de um *record* são sempre `private final`, com *getters* gerados automaticamente):

```java
public record Coordenada(double latitude, double longitude) {}
```

Isso não substitui classes "tradicionais" — objetos com comportamento rico e estado mutável continuam sendo classes normais, como `ContaBancaria` acima. Mas para o caso comum de "só um pacote de dados imutável", *record* é hoje preferível a escrever uma classe manualmente.

## 🧪 Atividade prática

Implemente um pequeno sistema de simulação bancária com uma classe `Conta` (atributos privados `saldo` e `limite`) e uma classe `Banco`, responsável por criar contas e operá-las — sacar, depositar, consultar saldo, consultar e atualizar limite. No `main`, instancie um `Banco`, crie algumas contas e exercite essas operações.

Em seguida, revisite sua solução respondendo:

1. Que atributos e métodos você tornou `private`? Por quê?
2. Se `Banco` precisasse impedir saque acima do limite, onde essa regra deveria morar — em `Conta` ou em `Banco`? Justifique em termos de coesão.

## 📝 Atividades

1. Defina, com suas palavras, a diferença entre objeto e classe — e explique por que "dados com métodos" é uma definição insuficiente de objeto.
2. O que é encapsulamento, e quais os seus benefícios? Exemplifique com um caso onde a ausência de encapsulamento causaria um bug real.
3. Modele, em Java, um `record ContatoImutavel(String nome, String email)` e uma classe mutável `Contato` equivalente com os mesmos dados. Discuta quando cada uma seria mais apropriada.
4. Pesquise: por que dizemos que orientação a objetos = TAD + herança + polimorfismo? O que a herança e o polimorfismo acrescentam que um TAD isolado não oferece?

## 🔗 Para saber mais

* Wegner, P. — *Concepts and Paradigms of Object-Oriented Programming* (OOPSLA 1989)
* Kay, A. — [*The Early History of Smalltalk*](http://worrydream.com/EarlyHistoryOfSmalltalk/)
* [JEP 395 — Records](https://openjdk.org/jeps/395)
