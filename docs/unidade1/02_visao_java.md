# Cap 2: Panorama da Linguagem Java

Antes de mergulhar em sintaxe, vale aplicar à própria linguagem Java o checklist de perguntas que fechou o capítulo anterior: de onde ela veio, que problema resolveu, e como se posiciona hoje frente a outras linguagens. As respostas ao checklist estão resumidas na seção "Paradigmas suportados".

## 🕰️ Breve histórico

Java nasceu em junho de 1991 como um projeto interno da Sun Microsystems chamado **"Oak"**, liderado por James Gosling. O objetivo original não era nem web nem servidores corporativos — era eletrônica de consumo: uma linguagem e uma máquina virtual que pudessem rodar de forma segura e portável em dispositivos variados, com sintaxe parecida com C, porém mais simples e mais uniforme (fortemente influenciada por C++, mas evitando várias de suas armadilhas).

A linguagem foi anunciada publicamente em 1995, e a versão **Java 1.0** foi lançada em janeiro de 1996, sob o lema **"Write Once, Run Anywhere"** — um programa Java compilado roda em qualquer plataforma que tenha uma JVM, sem recompilação. O modelo de segurança configurável (sandbox de *applets*) fez a linguagem decolar junto com a explosão da web nos anos 1990.

Alguns marcos importantes da evolução da linguagem:

| Versão | Ano | Principais novidades |
|---|---|---|
| Java 1.2 ("Java 2") | 1998 | Coleções, Swing, JIT |
| Java 5 | 2004 | **Generics**, enums, anotações, autoboxing, `for-each` |
| Java 8 | 2014 | **Lambdas**, Stream API, `Optional`, métodos `default` em interfaces |
| Java 9 | 2017 | Sistema de módulos (JPMS), início do ritmo de release semestral |
| Java 10 | 2018 | `var` para inferência de tipo local |
| Java 11 | 2018 | LTS; novo `HttpClient`; execução direta de arquivos `.java` |
| Java 17 | 2021 | LTS; **sealed classes** (*records* e *pattern matching* de `instanceof` já eram definitivos desde o Java 16) |
| **Java 21** | 2023 | **LTS**; **virtual threads** (Project Loom), pattern matching para `switch`, *record patterns* |
| Java 25 | 2025 | LTS mais recente; *scoped values* finalizados (*structured concurrency* segue em *preview*) |

Desde a versão 9, a Oracle adotou um **ritmo de release semestral** (março e setembro), com uma versão **LTS (Long-Term Support)** a cada dois anos — atualmente Java 17, 21 e 25. Este livro usa **Java 21 LTS** como base para todos os exemplos, por ser a versão LTS mais amplamente adotada em produção no momento em que este material foi escrito, já incorporando recursos modernos como *records*, `var`, *pattern matching* e *virtual threads*.

!!! tip "Java não parou no 8"
    É comum encontrar material didático (e até profissionais experientes) tratando "Java 8" como sinônimo de "Java moderno". Não é mais o caso: de 2017 para cá a linguagem incorporou *records*, *sealed classes*, *pattern matching*, *text blocks* e *virtual threads* — mudanças que afetam como se escreve Java idiomático hoje. Ao longo deste livro, vamos preferir esse estilo mais recente sempre que ele deixar o código mais claro, sem abrir mão de explicar a forma "clássica" quando ela ainda for relevante (por exemplo, em bases de código legadas).

## 🎯 Cinco princípios de design

Os objetivos originais de Gosling para a linguagem continuam sendo uma boa lente para entender decisões de design do Java até hoje:

1. Deve ser **orientada a objetos**.
2. Deve permitir que o **mesmo programa execute em vários sistemas operacionais** sem recompilação.
3. Deve ter **suporte nativo a redes** de computadores.
4. Deve ser projetada para **executar código remoto com segurança**.
5. Deve ser **fácil de usar**, aproveitando os melhores recursos de outras linguagens orientadas a objetos.

## 🧱 A plataforma Java

Uma fonte comum de confusão para quem começa em Java é que "Java" não é só a linguagem — é uma **plataforma** inteira, o que também explica parte da sua fama de ser "complexa":

* **JVM (Java Virtual Machine)**: a máquina virtual que executa o bytecode `.class`, incluindo o compilador **JIT** (*Just-In-Time*), responsável por otimizar trechos de bytecode para código de máquina nativo em tempo de execução.
* **JRE (Java Runtime Environment)**: JVM + bibliotecas necessárias para *executar* aplicações Java. Hoje, na prática, a distinção JRE/JDK separada praticamente desapareceu — desde o Java 11 a Oracle não distribui mais um JRE separado; instala-se o JDK mesmo apenas para rodar aplicações.
* **JDK (Java Development Kit)**: JRE + ferramentas para *desenvolver* — compilador (`javac`), depurador, `jshell` (REPL interativo, desde o Java 9) e utilitários de empacotamento.

Dentro do JDK, distinguem-se diferentes edições por público-alvo:

* **Java SE (Standard Edition)**: a base da plataforma — JVM, bibliotecas fundamentais (I/O, `String`, coleções). É o que este livro cobre.
* **Jakarta EE** (antiga *Java EE*, renomeada após a doação à Eclipse Foundation em 2017): edição voltada a aplicações corporativas e web, hoje sustentada por frameworks como Spring e Quarkus mais do que pela especificação EE em si.
* **Java ME (Micro Edition)**: voltada a dispositivos embarcados; hoje tem uso residual, dado que a maioria dos dispositivos móveis atuais usa outras plataformas (Android usa sua própria toolchain, não a JVM tradicional).

!!! note "OpenJDK vs. Oracle JDK"
    Hoje a forma mais comum de obter o JDK é via **OpenJDK**, a implementação de referência open source (licença GPLv2 + Classpath Exception), distribuída por vários fornecedores (Eclipse Temurin, Amazon Corretto, Azul Zulu, entre outros) — em vez do JDK proprietário da Oracle, cujos termos de licenciamento comercial mudaram diversas vezes nos últimos anos. Para desenvolvimento e a maior parte dos usos em produção, um OpenJDK é a escolha mais simples e sem restrições de licença.

## 💻 Instalando o ambiente

A forma mais prática de instalar e gerenciar versões do JDK hoje é por um **gerenciador de versões**, em vez de baixar instaladores manualmente — principalmente porque em algum momento você vai precisar alternar entre versões diferentes (um curso pedindo Java 21, um projeto legado em Java 8...).

=== "SDKMAN! (Linux/macOS/WSL)"
    ```bash
    curl -s "https://get.sdkman.io" | bash
    source "$HOME/.sdkman/bin/sdkman-init.sh"

    sdk list java                  # mostra os identificadores disponíveis
    # escolha um Eclipse Temurin (sufixo -tem) da linha 21, trocando 21.x.y pela versão listada
    sdk install java 21.x.y-tem
    sdk use java 21.x.y-tem
    java -version
    ```

=== "Windows"
    Recomenda-se o [Eclipse Temurin](https://adoptium.net/) (instalador `.msi`) ou o gerenciador de pacotes `winget`:
    ```powershell
    winget install EclipseAdoptium.Temurin.21.JDK
    ```

Para desenvolvimento sério, um **IDE** é altamente recomendado desde o início — IntelliJ IDEA (Community Edition, gratuita) ou VS Code com o *Extension Pack for Java* são as opções mais usadas hoje.

## 👋 Primeiro programa

O clássico "Hello World" em Java:

```java title="Hello.java"
public class Hello {
    public static void main(String[] args) {
        System.out.println("Olá, mundo!");
    }
}
```

```bash
javac Hello.java   # compila para Hello.class (bytecode)
java Hello          # executa na JVM
```

Desde o Java 11, para scripts rápidos e aprendizado, também é possível **executar um arquivo `.java` diretamente**, sem compilar explicitamente (o `java` compila em memória e roda na hora):

```bash
java Hello.java
```

E, para experimentar trechos de código sem nem criar um arquivo, o Java tem um REPL desde a versão 9:

```bash
$ jshell
jshell> System.out.println("Olá, mundo!")
Olá, mundo!
```

## 🧬 Estrutura básica da linguagem

A sintaxe de Java foi fortemente influenciada por C/C++ — chaves para blocos, ponto e vírgula ao final de comandos, tipos declarados antes dos identificadores. As diferenças mais importantes em relação a C++:

* Java **não é uma linguagem híbrida** no sentido em que C++ é: C++ preserva o C e permite funções e variáveis soltas, fora de classes; em Java, o código fica dentro de classes e de outros tipos. Não confunda com a execução "híbrida" (*bytecode* + máquina virtual) do Capítulo 1.
* Java tem **coletor de lixo (garbage collector)** integrado: não existe `free`/`delete` manual.
* Um programa Java é, estruturalmente, uma **coleção de classes que se relacionam**, cada uma agrupando atributos e métodos — o que já antecipa o assunto central da Unidade 2.

## 🧩 Paradigmas suportados

Java foi projetada primariamente em torno do paradigma **orientado a objetos baseado em classes** — é, e continua sendo, o núcleo da linguagem e o foco deste livro. Mas, especialmente a partir do Java 8, a linguagem passou a incorporar fortemente elementos de outros paradigmas:

* **Programação genérica**, via *generics* (desde o Java 5).
* **Programação funcional**, via lambdas, referências a métodos e a Stream API (Unidade 3, capítulo "Java Funcional").
* **Programação reflexiva**, via a API de *reflection* (Unidade 3, capítulo "Reflexão").
* **Programação concorrente**, historicamente via `Thread`/`ExecutorService`, e desde o Java 21 também via **virtual threads** (*Project Loom*) — threads leves gerenciadas pela JVM, que tornam código concorrente bloqueante barato de escalar sem reescrever tudo em estilo assíncrono.

### Respondendo ao checklist do Capítulo 1

| Pergunta | Java |
|---|---|
| Sistema de tipos | Estático e forte, com inferência local (`var`, desde o Java 10) |
| Paradigma predominante | Orientado a objetos, incorporando programação genérica, funcional, reflexiva e concorrente |
| *Class-based* ou *prototype-based*? | *Class-based* |
| Casamento de padrões | Sim: `instanceof` (Java 16), `switch` e *record patterns* (Java 21) |
| Funções são valores de primeira classe? | Métodos não são valores; lambdas e referências a métodos são objetos de interfaces funcionais, que podem ser passados, retornados e armazenados |

## ⚖️ Comparações com outras linguagens

Comparar Java com linguagens vizinhas ajuda a fixar suas decisões de design — vamos revisitar vários desses pontos com mais profundidade ao longo do livro.

### Java × C

| | Java | C |
|---|---|---|
| Paradigma | Orientado a objetos | Procedural |
| Distribuição | Bytecode portável (`.class`) | Precisa recompilar por plataforma |
| Ponteiros | Não existem (nem aritmética de ponteiros) | Centrais na linguagem |
| Memória | Gerenciamento automático (GC) | Manual (`malloc`/`free`) |
| Pré-processador | Não tem | Tem (`#define`, `#include`) |

### Java × C++

| | Java | C++ |
|---|---|---|
| Modelo de objetos | Simplificado | Mais complexo e flexível |
| Despacho de métodos | Virtual por padrão | Só virtual se declarado `virtual` |
| Passagem de argumentos | Sempre por valor (mas o valor pode ser uma referência a objeto) | Por valor, referência ou ponteiro |
| Herança múltipla | Não (mas há herança múltipla de *interfaces*) | Sim, de classes |
| Genéricos/templates | *Generics* (com *type erasure*, menos poderosos, mas mais seguros) | *Templates* (mais poderosos, mais complexos) |
| Sobrecarga de operadores | Não | Sim |

### Java × PHP

* Java é estaticamente tipada; PHP é dinamicamente tipada (embora o PHP moderno, 8.x, tenha ganhado tipagem gradual opcional).
* Java é de propósito geral; PHP é majoritariamente usado para desenvolvimento web.
* Ambas suportam concorrência hoje — Java nativamente desde o início, PHP de forma mais limitada e recente (fibers desde o PHP 8.1).

### Java × JavaScript

* Java é estaticamente tipada; JavaScript é dinamicamente tipada (TypeScript adiciona tipagem estática opcional por cima do JS).
* Java usa objetos baseados em **classes**; JavaScript nasceu baseado em **protótipos**, mas desde o ES6 (2015) oferece sintaxe de `class` (que ainda é, por baixo, protótipos).
* Java tem encapsulamento real via modificadores de acesso (`private`, `protected`); campos privados em classes JavaScript só se tornaram nativos com `#campo` no ES2022.
* Java é multithreaded nativamente; JavaScript é single-threaded com um *event loop* (concorrência via `async`/`await`, Web Workers ou, no Node.js, `worker_threads`).

## 📝 Atividades

1. Instale um JDK 21 usando SDKMAN! (ou equivalente na sua plataforma) e confirme a versão com `java -version`.
2. Execute o Hello World de duas formas: compilando com `javac` + `java`, e diretamente com `java Hello.java`.
3. Abra o `jshell` e experimente declarar uma variável com `var`, uma lista com `List.of(...)`, e imprimir seus elementos com um `for-each`.
4. Pesquise: o que é *type erasure* em generics Java, e por que ele é frequentemente citado como uma limitação em relação aos templates de C++?

## 🔗 Para saber mais

* [Documentação oficial do Java SE 21](https://docs.oracle.com/en/java/javase/21/)
* [OpenJDK](https://openjdk.org/) — implementação de referência open source
* [Adoptium / Eclipse Temurin](https://adoptium.net/) — distribuições de OpenJDK prontas para uso
