# Cap 12: Persistência com JDBC

## 🗄️ O que é o JDBC

Armazenar e recuperar dados de forma eficiente é, por si só, um desafio de engenharia considerável — e a maioria das aplicações não deveria reinventá-lo. **Persistência** é um serviço oferecido por diversos sistemas de banco de dados (Oracle, PostgreSQL, MySQL, SQLite...), e o Java provê uma API padronizada de acesso a esses sistemas: o **JDBC (Java Database Connectivity)**.

O JDBC é definido inteiramente como um conjunto de **interfaces** (`Connection`, `Statement`, `PreparedStatement`, `ResultSet`) — cada fabricante de banco de dados fornece um **driver** que implementa essas interfaces para o seu produto específico. É um exemplo direto e concreto de por que interfaces importam tanto (Capítulo 8): o mesmo código Java, escrito contra as interfaces do JDBC, funciona com PostgreSQL, MySQL ou SQLite — bastando trocar o driver e a *connection string*.

## 🚀 Passo a passo

Para os exemplos abaixo, vamos usar **SQLite**, por não exigir instalação de um servidor de banco separado — o banco inteiro vive em um único arquivo, o que o torna prático para aprendizado (a versão original deste material usava Derby, com o mesmo objetivo; SQLite é hoje a escolha mais comum para esse mesmo propósito didático).

```xml
<!-- pom.xml, se estiver usando Maven -->
<dependency>
    <groupId>org.xerial</groupId>
    <artifactId>sqlite-jdbc</artifactId>
    <version>3.46.1.0</version>
</dependency>
```

### Abrindo uma conexão

```java
String url = "jdbc:sqlite:agenda.db";

try (Connection conn = DriverManager.getConnection(url)) {
    System.out.println("conectado");
} catch (SQLException e) {
    throw new RuntimeException(e);
}
```

!!! tip "`try-with-resources` também vale para conexões de banco"
    `Connection`, `PreparedStatement` e `ResultSet` implementam `AutoCloseable` — o mecanismo visto no capítulo anterior. Usar `try-with-resources` garante que a conexão é fechada mesmo se uma exceção for lançada no meio do caminho, evitando vazamento de conexões — um bug de produção clássico em aplicações que gerenciam esses recursos manualmente.

### Criando uma tabela

```sql
CREATE TABLE contato (email VARCHAR(30), telefone VARCHAR(30), nome VARCHAR(30));
```

```java
String sql = """
    CREATE TABLE contato (
        email VARCHAR(30),
        telefone VARCHAR(30),
        nome VARCHAR(30)
    )
    """; // text block (Java 15+) — muito mais legível que concatenar strings com +

try (Connection conn = DriverManager.getConnection(url);
     PreparedStatement stmt = conn.prepareStatement(sql)) {
    stmt.execute();
}
```

### Inserindo dados — e por que usar `PreparedStatement`

```java
String sql = "INSERT INTO contato (nome, email, telefone) VALUES (?, ?, ?)";

try (Connection conn = DriverManager.getConnection(url);
     PreparedStatement stmt = conn.prepareStatement(sql)) {

    stmt.setString(1, "Sergio Costa");
    stmt.setString(2, "sergio@ufma.br");
    stmt.setString(3, "'); DELETE FROM contato; --");
    stmt.execute();

    System.out.println("inserido");
}
```

!!! danger "Esse terceiro valor não é um erro de exemplo — é a demonstração de por que `PreparedStatement` existe"
    Repare que o valor do telefone contém, propositalmente, um trecho de SQL malicioso: `'); DELETE FROM contato; --`. Imagine que o programa montasse o comando concatenando strings:

    ```java
    String sql = "INSERT INTO contato (nome, email, telefone) VALUES ('"
        + nome + "', '" + email + "', '" + telefone + "')";
    ```

    O texto enviado ao banco seria este:

    ```sql
    INSERT INTO contato (nome, email, telefone) VALUES ('Sergio Costa', 'sergio@ufma.br', ''); DELETE FROM contato; --')
    ```

    A aspa no começo do valor **fecha a string**, o `);` encerra o `INSERT`, o `DELETE` vira um segundo comando e o `--` transforma o resto em comentário. Se o driver aceitar mais de um comando na mesma chamada, a tabela é apagada. Recusar comandos múltiplos não resolve o problema: o atacante continua podendo alterar o sentido do comando original. Essa é a vulnerabilidade clássica de **injeção de SQL**. Com `PreparedStatement`, cada `?` é um **parâmetro de dado**, nunca interpretado como SQL: o valor é gravado literalmente, e o `DELETE` não é executado. **Nunca** construa SQL concatenando strings vindas do usuário.

### Consultando dados

```java
String sql = "SELECT * FROM contato";

try (Connection conn = DriverManager.getConnection(url);
     PreparedStatement stmt = conn.prepareStatement(sql);
     ResultSet rs = stmt.executeQuery()) {

    while (rs.next()) {
        System.out.println(rs.getString("nome"));
        System.out.println(rs.getString("email"));
    }
}
```

`ResultSet` é, mais uma vez, um exemplo do padrão **Iterator** (Capítulo 9 e 11): `rs.next()` avança o cursor para a próxima linha e retorna `false` quando não há mais dados — a mesma ideia de `Iterator.hasNext()`/`next()`, adaptada ao contexto de um cursor de banco de dados.

## 🧱 Um passo além: mapeando linhas para objetos

Juntando JDBC com o que já vimos sobre classes e coleções, é comum escrever um pequeno método que converte cada linha do `ResultSet` em um objeto de domínio — o embrião manual do que frameworks como JPA/Hibernate automatizam:

```java
public record Contato(String nome, String email, String telefone) {}

public List<Contato> listarContatos(Connection conn) throws SQLException {
    String sql = "SELECT nome, email, telefone FROM contato";
    List<Contato> resultado = new ArrayList<>();

    try (PreparedStatement stmt = conn.prepareStatement(sql);
         ResultSet rs = stmt.executeQuery()) {
        while (rs.next()) {
            resultado.add(new Contato(
                rs.getString("nome"),
                rs.getString("email"),
                rs.getString("telefone")
            ));
        }
    }
    return resultado;
}
```

!!! note "JDBC puro × ORMs"
    Em aplicações profissionais de maior porte, é raro escrever SQL cru manualmente para tudo — frameworks de mapeamento objeto-relacional (**ORM**), como JPA/Hibernate ou bibliotecas mais leves como jOOQ, automatizam boa parte desse trabalho de conversão. Mas eles são construídos **sobre** o JDBC, e entender a camada de baixo nível ajuda bastante a diagnosticar problemas de desempenho ou comportamento inesperado quando a camada de abstração "vaza" (o famoso [*leaky abstraction*](https://www.joelonsoftware.com/2002/11/11/the-law-of-leaky-abstractions/)).

## 📝 Atividades

1. Monte um pequeno CRUD completo (create, read, update, delete) para a tabela `contato`, usando SQLite e `PreparedStatement` em todas as operações.
2. Explique, com suas próprias palavras e sem usar jargão, por que `PreparedStatement` previne injeção de SQL enquanto concatenação de strings não previne.
3. Escreva o método `listarContatos` deste capítulo usando `try-with-resources` para `Connection`, `PreparedStatement` e `ResultSet` simultaneamente (os três podem ser abertos no mesmo bloco `try (...)`, separados por `;`).
4. Pesquise: o que é *connection pooling*, e por que abrir uma nova `Connection` a cada operação (como nos exemplos deste capítulo) é considerado ineficiente em uma aplicação real?

## 🔗 Para saber mais

* [Documentação oficial do JDBC](https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/)
* [SQLite JDBC Driver (xerial)](https://github.com/xerial/sqlite-jdbc)
* [OWASP — SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
