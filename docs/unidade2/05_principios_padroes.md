# Cap 9: Princípios e Padrões Orientados a Objetos

Este conteúdo é normalmente aprofundado em disciplinas específicas de projeto e arquitetura de software. Aqui, o objetivo é apenas apresentar o vocabulário — princípios e padrões mais citados no dia a dia de quem programa orientado a objetos — como preparação para estudos futuros mais completos.

## 🏛️ Princípios de projeto orientado a objetos

O conjunto de princípios mais conhecido é o acrônimo **SOLID**, popularizado por Robert C. Martin a partir de ideias já discutidas por autores como Bertrand Meyer e Barbara Liskov nas décadas anteriores. Cada letra nomeia um princípio independente — todos voltados a reduzir acoplamento e aumentar a capacidade de um sistema evoluir sem quebrar:

* **S — Single Responsibility Principle (Princípio da Responsabilidade Única).** Uma classe deve ter um, e apenas um, motivo para mudar. Conecta diretamente com a ideia de **coesão** vista no Capítulo 5: uma classe `Pedido` que calcula totais, envia e-mails de confirmação e grava no banco de dados tem três motivos diferentes para mudar — e deveria, provavelmente, ser três classes.

* **O — Open/Closed Principle (Princípio Aberto/Fechado).** Entidades de software devem estar abertas para **extensão**, mas fechadas para **modificação**. Voltando ao exemplo de `Shape` do Capítulo 8: adicionar uma nova forma geométrica não deveria exigir alterar o código que já processa `List<Shape>` — só criar uma nova subclasse ou implementação.

* **L — Liskov Substitution Principle (Princípio da Substituição de Liskov).** Formulado por Barbara Liskov em 1987: se `S` é subtipo de `T`, então objetos de tipo `T` podem ser substituídos por objetos de tipo `S` sem alterar a corretude do programa. É uma forma mais rigorosa de dizer que o polimorfismo de inclusão (Capítulo 7) só "funciona de verdade" se a subclasse realmente respeitar o contrato da superclasse — não bastando apenas compilar.

    !!! example "O clássico contra-exemplo: Quadrado não deveria herdar de Retângulo"
        Um `Quadrado` parece, geometricamente, um caso especial de `Retângulo` (largura = altura). Mas se `Retangulo` tem `setLargura`/`setAltura` independentes, um `Quadrado extends Retangulo` precisa sobrescrever ambos para manter a invariante "largura = altura" — e isso quebra o comportamento esperado por qualquer código que já use `Retangulo` (por exemplo, um teste que assume `setLargura(5)` não altera a altura). O tipo compila, mas viola LSP.

* **I — Interface Segregation Principle (Princípio da Segregação de Interfaces).** Nenhum cliente deve ser forçado a depender de métodos que não usa. Prefira várias interfaces pequenas e específicas a uma única interface "gorda" — o Capítulo 8 já deu a ferramenta certa para isso: como uma classe Java pode implementar quantas interfaces quiser, não há custo em separá-las.

* **D — Dependency Inversion Principle (Princípio da Inversão de Dependência).** Módulos de alto nível não devem depender de módulos de baixo nível — ambos devem depender de **abstrações** (interfaces). Por exemplo, uma classe `RelatorioService` não deveria depender diretamente de uma classe concreta `MySQLRepositorio`, e sim de uma interface `Repositorio`, cuja implementação concreta é fornecida de fora (esse é, inclusive, o princípio por trás de frameworks de *injeção de dependência* como o Spring, amplamente usado em Java).

Além do SOLID, dois princípios mais antigos, mas igualmente centrais, já apareceram implicitamente nos capítulos anteriores e vale nomear explicitamente:

* **DRY (Don't Repeat Yourself)**: todo conhecimento deve ter uma representação única e não ambígua no sistema — é parte da motivação por trás de herança, generics e métodos `default` de interface.
* **Prefira composição a herança**: já discutido no Capítulo 6 — herança cria um acoplamento muito forte (a subclasse depende de detalhes de implementação da superclasse); composição, ao delegar comportamento a um objeto membro, tende a ser mais flexível.

## 🧱 Padrões de Projeto (Design Patterns)

Um **padrão de projeto** é uma solução reutilizável para um problema recorrente de design de software — não é código pronto para copiar, mas uma estrutura de relação entre classes/objetos que se repete em contextos diferentes. O catálogo mais influente é o do livro *Design Patterns: Elements of Reusable Object-Oriented Software* (1994), dos autores conhecidos como **"Gang of Four" (GoF)**: Gamma, Helm, Johnson e Vlissides.

Os padrões GoF se dividem em três categorias:

### Padrões Criacionais
Tratam da criação de objetos, abstraindo o processo de instanciação.

* **Factory Method**: delega a criação de objetos a subclasses, em vez de usar `new` diretamente — útil quando a classe exata a instanciar só é conhecida em tempo de execução.
* **Singleton**: garante que uma classe tenha uma única instância, com um ponto de acesso global — usado (às vezes em excesso) para recursos compartilhados como conexões de configuração.
* **Builder**: separa a construção de um objeto complexo (com muitos parâmetros opcionais) de sua representação final — em Java, é comum ver isso implementado manualmente ou com bibliotecas como Lombok (`@Builder`).

### Padrões Estruturais
Tratam de como classes e objetos se compõem para formar estruturas maiores.

* **Adapter**: converte a interface de uma classe em outra interface esperada pelo cliente — útil para integrar bibliotecas externas sem modificá-las.
* **Decorator**: adiciona responsabilidades a um objeto dinamicamente, sem alterar sua classe — as classes de I/O de Java (`BufferedInputStream` envolvendo um `InputStream`) são o exemplo clássico do próprio JDK.
* **Composite**: compõe objetos em estruturas de árvore para representar hierarquias parte-todo, tratando objetos individuais e composições de forma uniforme.

### Padrões Comportamentais
Tratam de como objetos colaboram e distribuem responsabilidades.

* **Strategy**: encapsula uma família de algoritmos intercambiáveis por trás de uma interface comum — em Java moderno, frequentemente substituído por uma simples interface funcional + lambda (Unidade 3).
* **Observer**: define uma dependência um-para-muitos entre objetos, de forma que quando um objeto muda de estado, todos os seus dependentes são notificados — base de praticamente todo *framework* de interface gráfica, incluindo o Swing que veremos na Unidade 3.
* **Iterator**: fornece uma forma de acessar sequencialmente os elementos de uma coleção sem expor sua representação interna — é, literalmente, a interface `Iterator` do Java, por trás de todo `for-each` (Capítulo 4 e Unidade 3, Coleções).

!!! tip "Você já usou vários desses padrões sem perceber"
    Boa parte do valor de aprender esse vocabulário não é "decorar a receita" de cada padrão, mas ganhar um nome compartilhado para estruturas que você já usa (ou vai usar) intuitivamente. Ao longo da Unidade 3 você vai reencontrar Iterator (Coleções), Observer (Swing) e Strategy (Java Funcional) como parte natural das próprias bibliotecas do JDK — não como exercício isolado de "padrões de projeto".

## 📝 Atividades

1. Escolha uma classe que você já implementou em um exercício anterior deste livro (por exemplo, `ContaBancaria` ou `SocialNetwork`) e avalie: ela respeita o princípio da responsabilidade única? Se não, como você a dividiria?
2. Explique, com um exemplo próprio (não o de `Quadrado`/`Retangulo`), uma situação em que uma hierarquia de herança aparentemente correta viola o Princípio de Substituição de Liskov.
3. Identifique, no JDK ou em alguma biblioteca Java que você já usou, um exemplo real de padrão Decorator, Observer ou Strategy, e explique como ele se encaixa na definição do padrão.
4. Pesquise a diferença entre Factory Method e o padrão Abstract Factory — em que situação um seria preferível ao outro?

## 🔗 Para saber mais

* Gamma, Helm, Johnson & Vlissides — *Design Patterns: Elements of Reusable Object-Oriented Software* (1994)
* Martin, R. C. — *Clean Architecture* / artigos originais sobre os princípios SOLID
* Terra, R. et al. — [*Engenharia de Software Moderna*](https://engsoftmoderna.info/) — capítulos 5 (Princípios) e 6 (Padrões de Projeto), livro digital gratuito
* [Refactoring.Guru — catálogo ilustrado de padrões de projeto](https://refactoring.guru/design-patterns)
