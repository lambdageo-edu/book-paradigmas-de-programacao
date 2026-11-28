# Cap 14: Reflexão

## 🪞 O que é reflexão

Reflexão é a capacidade de um programa **examinar a si mesmo** e o seu ambiente, e mudar seu comportamento de acordo com o que encontra. Mais precisamente: reflexão é o processo pelo qual um programa observa e modifica sua própria estrutura e comportamento, exigindo acesso à sua representação interna — os chamados **metadados**.

Isso torna reflexão um tipo de **metaprogramação**: o programa realiza computações a respeito de si mesmo. A diferença em relação a macros (outro mecanismo clássico de metaprogramação) é o **momento** em que isso acontece — macros operam em nível sintático, antes da execução; reflexão opera em **tempo de execução**.

Em programação orientada a objetos, os metadados são organizados em **metaobjetos**. A capacidade de acessá-los em tempo de execução é chamada **introspecção** — um subconjunto da reflexão, que permite ao programa obter informações sobre si mesmo. Para dar suporte a metaprogramação, uma linguagem precisa expor os elementos usados para codificar suas próprias entidades — no caso do Java, dados sobre classes, métodos e atributos de um objeto.

Java oferece **introspecção** ampla, mas uma capacidade **limitada** de modificação: pela API de reflexão é possível criar objetos, invocar métodos e ler ou escrever atributos, mas não acrescentar métodos ou atributos a uma classe já carregada, como se faz em Ruby ou Python.

## 🏷️ Anotações

Anotações são metadados adicionados a classes, atributos e métodos — expressam alguma intenção do programador em relação ao elemento anotado. Vale marcar a diferença em relação a comentários:

* Comentários existem apenas no código-fonte (o compilador os descarta na análise léxica); anotações podem permanecer no código compilado.
* Comentários são texto livre, difícil de processar automaticamente; anotações são estruturadas e projetadas justamente para processamento automatizado.

## 🐍 Motivação: o problema que reflexão resolve

Em linguagens dinamicamente tipadas como Python, é comum usar o conceito de **duck typing** — "se anda como um pato e faz quack como um pato, então é um pato". Considere três classes independentes (talvez de bibliotecas diferentes, escritas por autores diferentes) que coincidentemente têm um método `getNome`:

```python
class Docente:
    def __init__(self, nome):
        self.nome = nome
    def getNome(self):
        return self.nome

class Discente:
    def __init__(self, nome):
        self.nome = nome
    def getNome(self):
        return self.nome

class Tecnico:
    def __init__(self, nome):
        self.nome = nome
    def getNome(self):
        return self.nome
```

Como Python não checa tipos em tempo de compilação, uma função pode aplicar `getNome()` a qualquer objeto da lista, sem exigir que compartilhem uma superclasse comum — o único pré-requisito é que o método exista, verificado só em tempo de execução:

```python
def imprime_nomes(lista):
    for item in lista:
        print(item.getNome())
```

Em Java, essa abordagem **não compila**: uma variável do tipo `Object` não tem um método `getNome()` conhecido estaticamente.

```java
public static void imprimeNomes(List<Object> lista) {
    for (Object i : lista) {
        System.out.println(i.getNome()); // erro de compilação!
    }
}
```

Se as classes fossem escritas pelo mesmo time, a solução natural seria herança e polimorfismo (Capítulos 6 e 7). Mas quando isso não é possível — classes de terceiros, sem controle sobre o design — Java oferece **reflexão** como alternativa, ao custo de perder a checagem estática:

```java
public static void imprimeNomes(List<Object> lista) {
    for (Object i : lista) {
        Class<?> c = i.getClass();
        try {
            Method m = c.getMethod("getNome");
            String nome = (String) m.invoke(i);
            System.out.println(nome);
        } catch (ReflectiveOperationException e) {
            e.printStackTrace();
        }
    }
}
```

`c.getMethod("getNome")` procura, em tempo de execução, um método público chamado `getNome` na classe de `i`; `m.invoke(i)` o chama. Se o método não existir, o erro só aparece em execução — a mesma troca de confiabilidade por flexibilidade discutida no Capítulo 3 quando comparamos tipagem estática e dinâmica.

!!! warning "Reflexão tem custo — use com moderação"
    Chamadas via reflexão são significativamente mais lentas que chamadas diretas (o JIT, visto no Capítulo 2, otimiza muito pior código que passa por `Method.invoke`), e o próprio compilador perde a capacidade de checar erros de tipo e nome de método antecipadamente. Reflexão é uma ferramenta poderosa para **frameworks** — código genérico que precisa operar sobre tipos que só existirão depois, escritos por quem usa o framework — não uma ferramenta de uso cotidiano em lógica de negócio comum.

## 🏗️ Um exemplo mais realista: anotações + reflexão em conjunto

Retomando o exemplo de `JTable` do capítulo anterior: escrever manualmente uma `TableModel` para cada classe de domínio é repetitivo. Usando anotações e reflexão, é possível escrever **uma única** `TableModel` genérica que funciona para qualquer classe anotada.

Primeiro, uma anotação customizada para marcar quais atributos devem virar colunas:

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME) // precisa estar disponível em tempo de execução
public @interface Coluna {
    String nome();
    int posicao();
}
```

Aplicada a uma classe de domínio:

```java
public class Filme {
    @Coluna(nome = "Ano", posicao = 0)
    private String ano;

    @Coluna(nome = "Diretor", posicao = 1)
    private String diretor;

    @Coluna(nome = "Nome", posicao = 2)
    private String nome;

    // getters
}
```

Uma interface para abstrair a origem dos dados (o padrão DAO — *Data Access Object* — combinado com generics, visto no Capítulo 7):

```java
public interface RepositorioGenerico<T> {
    List<T> listar();
}
```

E, por fim, a `TableModel` genérica, que usa reflexão para descobrir colunas e valores em tempo de execução, a partir das anotações:

```java
public class TabelaGenerica<T> extends AbstractTableModel {
    private final Class<T> tipo;
    private final List<T> dados;

    public TabelaGenerica(Class<T> tipo, RepositorioGenerico<T> repositorio) {
        this.tipo = tipo;
        this.dados = repositorio.listar();
    }

    @Override
    public int getRowCount() {
        return dados.size();
    }

    @Override
    public int getColumnCount() {
        return (int) camposAnotados().count();
    }

    @Override
    public String getColumnName(int coluna) {
        return camposAnotados()
            .filter(f -> f.getAnnotation(Coluna.class).posicao() == coluna)
            .findFirst()
            .map(f -> f.getAnnotation(Coluna.class).nome())
            .orElse(null);
    }

    @Override
    public Object getValueAt(int linha, int coluna) {
        T objeto = dados.get(linha);
        Field campo = camposAnotados()
            .filter(f -> f.getAnnotation(Coluna.class).posicao() == coluna)
            .findFirst()
            .orElseThrow();
        try {
            campo.setAccessible(true); // permite ler campos private
            return campo.get(objeto);
        } catch (IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }

    private Stream<Field> camposAnotados() {
        return Arrays.stream(tipo.getDeclaredFields())
            .filter(f -> f.isAnnotationPresent(Coluna.class));
    }
}
```

O construtor recebe a classe (`new TabelaGenerica<>(Filme.class, repositorio)`) para que as colunas sejam conhecidas mesmo quando o repositório não devolve nenhum registro.

Com isso, **qualquer** classe de domínio anotada com `@Coluna` pode virar uma `JTable` sem escrever uma nova `TableModel` para cada uma — é exatamente esse tipo de generalização que sustenta frameworks amplamente usados em Java, como Spring (`@Autowired`, `@RequestMapping`) e Jakarta Persistence/Hibernate (`@Entity`, `@Column`): todos usam reflexão sobre anotações para conectar seu código a comportamento genérico da biblioteca, sem exigir que você escreva boilerplate repetitivo.

## 📝 Atividades

1. Implemente a classe `TabelaGenerica` deste capítulo (ou uma versão simplificada) e use-a com uma classe `Filme` anotada, exibindo os dados em uma `JTable` real.
2. Explique, com suas palavras, a diferença entre introspecção e reflexão — dado que o texto afirma que Java oferece introspecção ampla, mas só uma capacidade limitada de modificação.
3. Pesquise: como a anotação `@Override`, que você já usou em capítulos anteriores, é diferente de `@Coluna` deste capítulo? (Dica: pesquise sobre `RetentionPolicy.SOURCE` vs. `RUNTIME`.)
4. Compare o exemplo de *duck typing* em Python com a solução via reflexão em Java — em que sentido reflexão é uma forma de "recuperar", em uma linguagem estaticamente tipada, parte da flexibilidade que uma linguagem dinamicamente tipada tem por padrão?

## 🔗 Para saber mais

* Forman, I. & Forman, N. — *Java Reflection in Action*
* [Documentação oficial: The Reflection API (Oracle)](https://docs.oracle.com/javase/tutorial/reflect/)
* Pesquise sobre metaprogramação em Ruby, para comparar com o modelo de reflexão do Java (Ruby permite reflexão muito mais ampla, incluindo modificar classes existentes em tempo de execução — o chamado *monkey patching*).
