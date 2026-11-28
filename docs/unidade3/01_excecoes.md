# Cap 10: Tratamento de Exceções

Retomando o critério de **confiabilidade** do Capítulo 1: tratamento de exceções é um dos mecanismos de linguagem que mais contribui para confiabilidade, porque permite interceptar condições de erro — e, em certos casos, **obriga** o programador a lidar com elas — em vez de deixar o programa simplesmente continuar em um estado inconsistente ou falhar silenciosamente.

## 🌳 Hierarquia de exceções em Java

Toda condição excepcional em Java é representada por uma subclasse de `Throwable`, que tem duas subclasses diretas: `Error` e `Exception`. Como `RuntimeException` (subclasse de `Exception`) tem regras próprias, na prática distinguimos três grupos:

* **`Error`** — condições graves, geralmente fora do controle do programa (falta de memória, erro interno da JVM). Não se espera que sejam tratadas.
* **`RuntimeException`** — erros de **lógica de programação** que o programador deveria ter evitado: índice de array fora dos limites, divisão por zero, chamar um método em uma referência `null`. São as chamadas **exceções não verificadas** (*unchecked*).
* **`Exception`** (excluindo `RuntimeException`) — condições do **ambiente de execução**, fora do controle direto do programador, mas recuperáveis: arquivo não encontrado, rede indisponível, URL malformada. São **exceções verificadas** (*checked*) — o compilador exige que sejam tratadas ou declaradas.

```java
public class Zero {
    public static void main(String[] args) {
        int numerador = 10;
        int denominador = 0;
        System.out.println(numerador / denominador); // lança ArithmeticException (unchecked)
    }
}
```

Seria impraticável exigir `try`/`catch` para toda `RuntimeException` possível — por isso o compilador não obriga isso. Já para exceções verificadas, um método precisa **tratar** todas que podem ocorrer no seu corpo ou **declará-las explicitamente**, repassando a responsabilidade para quem o chamar.

## 📣 Declarando exceções lançadas (`throws`)

A lista de exceções que um método pode lançar faz parte de sua **interface pública** — documenta, no próprio código, o que pode dar errado ao chamá-lo:

```java
public void meuMetodo() throws Excecao1, Excecao2 { /* ... */ }
```

Exemplos do próprio JDK:

```java
public static int parseInt(String s) throws NumberFormatException // em Integer
public String readLine() throws IOException                       // em BufferedReader
```

## 🚀 Lançando uma exceção (`throw`)

Basta instanciar a classe de exceção apropriada e lançá-la com `throw`:

```java
throw new IOException("Erro de E/S!");
```

## 🥅 Capturando uma exceção com `try`/`catch`

Se um método que abre um arquivo falha, sua execução é interrompida ali, e o controle passa para o bloco preparado para tratar esse tipo de falha:

```java
try {
    // código que pode lançar exceções
} catch (MinhaPrimeiraExcecao e) {
    // trata esta exceção específica
} catch (MinhaSegundaExcecao e) {
    // trata esta outra
}
// código não protegido continua daqui, se não houve exceção
// ou se ela foi capturada com sucesso
```

Regra prática de bolso: **capture** (`try`/`catch`) a exceção que você sabe processar; **propague** (`throws`) aquela que você não sabe resolver, delegando a responsabilidade para um nível acima.

Desde o Java 7, é possível capturar **vários tipos** de exceção no mesmo `catch`, quando o tratamento é idêntico:

```java
try {
    processar();
} catch (IOException | SQLException e) {
    log.error("Falha ao processar", e);
}
```

## 🧹 A cláusula `finally`

Imagine abrir um arquivo dentro de um `try` — em algum momento é preciso liberar esse recurso, com um "código de limpeza". Colocar essa limpeza no fim do `try` não resolve, porque uma exceção pode interromper a execução antes de chegar lá; duplicar em cada `catch` é repetitivo e ainda deixaria de rodar se nenhuma exceção fosse lançada. `finally` resolve isso: o bloco sempre executa, tenha ocorrido exceção ou não.

```java
try {
    ligaTorneira();
    molharGramado();
} finally {
    desligaTorneira(); // sempre executa
}
```

### Try-with-resources (desde o Java 7): a forma moderna de garantir limpeza

Para recursos que implementam `AutoCloseable` (a maioria das classes de I/O e conexões de banco), existe uma forma mais segura e concisa que `finally` manual: **try-with-resources**, que fecha o recurso automaticamente ao sair do bloco, mesmo em caso de exceção:

```java
try (OutputStream osf = new FileOutputStream("filename");
     OutputStream osb = new BufferedOutputStream(osf);
     ObjectOutput op = new ObjectOutputStream(osb)) {

    op.writeObject(writableObject);

} catch (IOException e) {
    System.out.println(e);
} // os três recursos são fechados automaticamente aqui, em ordem inversa à criação
```

!!! tip "Prefira try-with-resources a `finally` manual"
    Além de mais curto, try-with-resources evita um erro sutil e comum no `finally` manual: se `op.close()` dentro de um `finally` lançar sua própria exceção, ela pode mascarar a exceção original que causou a falha. Try-with-resources trata isso corretamente, preservando a exceção original como principal e anexando qualquer exceção de fechamento como "suprimida" (`getSuppressed()`).

## 🏦 Exemplo de aplicação: exceções customizadas

Retomando o exemplo de conta bancária dos capítulos anteriores: duas condições de erro são esperadas — saldo insuficiente e conta inexistente. O primeiro passo é criar uma classe de exceção para cada uma, estendendo `Exception` (para que sejam verificadas, e o compilador obrigue o tratamento):

```java
public class SaldoInsuficienteException extends Exception {
    private final double saldo;
    private final String numero;

    public SaldoInsuficienteException(double saldo, String numero) {
        super("Saldo insuficiente!");
        this.saldo = saldo;
        this.numero = numero;
    }

    public double getSaldo() { return saldo; }
    public String getNumero() { return numero; }
}

public class ContaNaoEncontradaException extends Exception {
    public ContaNaoEncontradaException(String numero) {
        super("A conta " + numero + " não existe");
    }
}
```

```java
class Conta {
    private String numero;
    private double saldo;

    public String getNumero() { return numero; }

    void debitar(double valor) throws SaldoInsuficienteException {
        if (valor <= saldo) {
            saldo -= valor;
        } else {
            throw new SaldoInsuficienteException(saldo, numero);
        }
    }
}

class Banco {
    private final List<Conta> contas = new ArrayList<>();

    private Conta procurar(String numero) throws ContaNaoEncontradaException {
        return contas.stream()
            .filter(c -> c.getNumero().equals(numero))
            .findFirst()
            .orElseThrow(() -> new ContaNaoEncontradaException(numero));
    }

    void debitar(String numero, double valor)
            throws SaldoInsuficienteException, ContaNaoEncontradaException {
        Conta c = procurar(numero);
        c.debitar(valor);
    }
}
```

```java
try {
    banco.debitar(numero, valor);
} catch (SaldoInsuficienteException e) {
    System.out.println(e.getMessage() + " Conta/saldo: " + e.getNumero() + "/" + e.getSaldo());
} catch (ContaNaoEncontradaException e) {
    System.out.println(e.getMessage());
}
```

## 🔀 Uma alternativa funcional: erros como valores

Vale antecipar algo que vamos retomar no capítulo de Java Funcional: exceções não são a única forma de modelar falhas. Linguagens funcionais frequentemente preferem representar um resultado que **pode falhar** como um **valor de retorno** — em vez de interromper o fluxo de controle — usando tipos como `Either`/`Result`. Essa abordagem, às vezes chamada de *programação orientada a trilhos* (*Railway Oriented Programming*), evita o custo de performance de lançar exceções e torna explícito, no tipo de retorno, que uma operação pode falhar. Java não tem um `Result` nativo, mas `Optional<T>` (Java 8+) cobre parcialmente o caso mais simples — "valor pode não existir" — como veremos adiante.

## 📝 Atividades

1. Explique a diferença entre exceções *checked* e *unchecked* em Java, e dê um exemplo de cada uma que não esteja neste capítulo.
2. Reescreva o exemplo de `Conta`/`Banco` usando `try-with-resources` para simular a abertura e fechamento de um "recurso de auditoria" (crie uma classe simples que implementa `AutoCloseable`).
3. Por que é considerado má prática capturar `Exception` (a classe genérica) em vez de tipos específicos? Dê um exemplo de bug que isso pode mascarar.
4. Pesquise como exceções são tratadas em Python (`try`/`except`) e compare com Java: Python distingue exceções *checked* e *unchecked*? Que implicações isso tem para quem projeta uma API?

## 🔗 Para saber mais

* [Exceções em Python — Real Python](https://realpython.com/python-exceptions/)
* [Tutorial oficial: the try-with-resources statement](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html)
* Sobre alternativas a exceções: pesquise "Railway Oriented Programming" e o tipo `Either` em Haskell. Em Elixir, linguagem dinâmica, a convenção equivalente são as tuplas `{:ok, valor}` e `{:error, motivo}`.
