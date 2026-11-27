# Cap 1: Paradigmas de Programação

## 🎯 Sobre esta disciplina

Esta disciplina tem quatro objetivos entrelaçados:

* Apresentar os principais **paradigmas** de linguagens de programação.
* Fazer um **estudo comparativo** entre eles — o que cada um resolve bem, e a que custo.
* Entender como paradigmas diferentes **se integram** nas linguagens modernas (poucas linguagens hoje são "puras" em um único paradigma).
* Aprofundar em uma linguagem concreta — neste livro, **Java** — como estudo de caso do paradigma orientado a objetos.

Ao final, o objetivo não é apenas "saber Java", mas ter vocabulário conceitual para avaliar *qualquer* linguagem nova que você encontrar ao longo da carreira: por que ela tomou certas decisões de design, que trade-offs isso implica, e como isso se compara com o que você já conhece.

!!! info "Bibliografia de referência"
    Sebesta, R. *Conceitos de Linguagens de Programação* (9ª ed.); Watt, D. *Programming Language Design Concepts*; Varejão, F. *Linguagens de Programação: Conceitos e Técnicas* (disponível gratuitamente online); Tucker, A. & Noonan, R. *Linguagens de Programação — Princípios e Paradigmas*.

---

## 🧩 O que é um programa, um algoritmo, uma linguagem?

Antes de comparar linguagens, vale fixar três definições que serão usadas o curso inteiro:

* **Programa**: uma sequência de símbolos que especifica uma computação.
* **Algoritmo**: um método finito, descrito em um vocabulário fixo e por instruções precisas, organizado em passos discretos, cuja execução não exige intuição ou criatividade — e que eventualmente termina.
* **Linguagem de programação**: um formalismo artificial no qual algoritmos podem ser expressos.

Essas três ideias parecem óbvias, mas a terceira é a mais importante para este curso: uma linguagem é, antes de tudo, uma **notação formal** — e notações diferentes tornam certos algoritmos mais fáceis ou mais difíceis de expressar. É exatamente essa diferença de notação, aplicada a um mesmo problema computacional, que chamamos de **paradigma**.

---

## 🕰️ Um breve histórico: das linguagens de baixo nível ao paradigma estruturado

### As linguagens de primeira geração

Os primeiros computadores com programa armazenado eram programados diretamente em **linguagem de máquina**, específica para cada arquitetura. Para tornar isso um pouco mais legível, surgiu o **Assembly**: símbolos mnemônicos mapeados 1-para-1 nas instruções do processador.

```text
; soma os valores n+...+3+2+1(+0) em um laço e
; guarda o resultado na posição de memória "sum"

loop:   S(x)->Ac+  n    ; carrega n no acumulador
        Cc->S(x)   pos  ; se AC >= 0, salta para pos
        halt             ; senão, termina
        .empty
pos:    S(x)->Ah+  sum  ; soma n a sum
        At->S(x)   sum
        S(x)->Ac+  n
        S(x)->Ah-  one   ; decrementa n
        At->S(x)   n
        Cu->S(x)   loop  ; volta ao início do laço
```

O grande problema dessas linguagens de **primeira geração** era a **falta de portabilidade**: um programa escrito para uma máquina não rodava em outra.

### As primeiras linguagens de alto nível

As linguagens de alto nível resolveram parte do problema de portabilidade, mas herdaram do assembly um controle de fluxo baseado em saltos condicionais e incondicionais (`goto`) — um mecanismo flexível, mas que produz código cujo fluxo de execução não acompanha a ordem em que o texto é lido. Isso prejudica legibilidade e manutenibilidade, e foi um dos fatores por trás da chamada "crise do software", no fim dos anos 1960.

As quatro linguagens fundadoras dessa era:

=== "Fortran (1957)"
    Criada por uma equipe da IBM liderada por John Backus, é considerada a primeira linguagem de alto nível, criada para permitir que fórmulas matemáticas fossem traduzidas automaticamente em código eficiente para o IBM 704. Ainda é usada por matemáticos, físicos e engenheiros — e continua evoluindo (a revisão mais recente do padrão é de 2023).

=== "Lisp (1958)"
    Criada por John McCarthy, é a segunda linguagem de alto nível e a primeira linguagem **funcional**, baseada no cálculo lambda. Sintaxe minimalista — tudo é lista — e altamente customizável via macros. É a ancestral de dialetos modernos como Scheme, Racket e **Clojure**, que estudaremos mais adiante no curso de Programação Funcional.

=== "Algol60 (1960)"
    Projetada por comitê (incluindo os criadores de Fortran e Lisp) com a ambição de ser uma "linguagem universal". Comercialmente não decolou, mas influenciou profundamente linguagens posteriores, entre elas C.

    ```algol
    procedure Absmax(a) Size:(n, m) Result:(y) Subscripts:(i, k);
        value n, m; array a; integer n, m, i, k; real y;
    comment Encontra o maior elemento em módulo da matriz a (n×m)
        e devolve seu valor em y e seus índices em i, k;
    begin
        integer p, q;
        y := 0; i := k := 1;
        for p := 1 step 1 until n do
            for q := 1 step 1 until m do
                if abs(a[p, q]) > y then
                    begin y := abs(a[p, q]);
                        i := p; k := q
                    end
    end Absmax
    ```

=== "Cobol (1959)"
    Introduziu estruturas de dados nomeadas e voltadas a processamento comercial, dominando o mercado de aplicações bancárias e de mainframe — onde, aliás, ainda está presente hoje.

---

## 🧠 O que é um paradigma de programação?

Um paradigma de programação é um **estilo distintivo de programar**, caracterizado pela predominância de certos conceitos-chave. Programas em assembly são, no fundo, todos "iguais" — sequências de instruções de processador. Mas assim que subimos para uma linguagem de alto nível, a forma de pensar o problema começa a divergir de acordo com o paradigma escolhido: as linguagens de um mesmo paradigma tendem a compartilhar muito mais entre si do que com linguagens de paradigmas diferentes.

Vamos adotar a classificação a seguir, que separaremos em dois grandes grupos:

### Paradigma Imperativo
Baseado no **processo de mudança de estado**: variáveis, valores e atribuições. É o modelo mental mais próximo da máquina — uma sequência de comandos que alteram a memória passo a passo.

* **Estruturado/procedimental**: refinamentos sucessivos, blocos aninhados de comandos, forte desestímulo ao uso de `goto` — apoiado nos três comandos fundamentais: sequência, seleção e iteração.
* **Orientado a objetos**: uma coleção de objetos que se relacionam e se comunicam por troca de mensagens — Smalltalk, C++, Java, Python, Ruby, Kotlin, C#.

### Paradigma Declarativo
Abstrai-se de **como** o computador executa a tarefa, especificando apenas **o que** deve ser calculado.

* **Funcional**: computação como composição e aplicação de funções — Haskell, Lisp, Clojure, Elixir.
* **Lógico**: uma coleção de fatos e regras sobre a qual se fazem perguntas — Prolog, Mercury.

!!! note "Por que só estes dois grandes grupos?"
    Existem outras classificações mais finas (paradigma orientado a eventos, reativo, concorrente...), mas para os fins comparativos deste curso, a divisão imperativo × declarativo — com objetos de um lado e funções/lógica do outro — é suficiente e é a que aparece na maior parte da literatura clássica (Watt, Sebesta, Van Roy).

### Programação Estruturada, em mais detalhe

O paradigma estruturado nasceu de uma reação coletiva contra o "código espaguete" produzido pelo uso indiscriminado de `goto`. Edsger Dijkstra e C.A.R. Hoare, entre outros, sintetizaram práticas que hoje consideramos óbvias: **desenvolvimento top-down**, **modularização**, **tipos estruturados** e o uso disciplinado de apenas três construções de controle — sequência, seleção e iteração — no lugar de saltos arbitrários.

Foi também nessa época que se consolidou o conceito de **tipo abstrato de dados (TAD)**: um conjunto de objetos e as operações definidas sobre eles, onde a especificação das operações define uma interface entre o TAD e o resto do programa — descrevendo *o que* as operações fazem, mas não *como* fazem. Esse conceito é o alicerce direto sobre o qual as classes de linguagens orientadas a objetos, como Java, foram construídas — algo que retomaremos na Unidade 2.

### Programação Funcional, em mais detalhe

Vale separar dois usos do termo "funcional", como propõe Martin Odersky (criador de Scala):

* **Sentido amplo**: programar com foco em funções e suas composições. Funções são **valores de primeira classe** — podem ser passadas como argumento, retornadas por outras funções, guardadas em estruturas de dados e usadas como literais anônimos (**lambdas**):

```haskell
Prelude> map (\x -> 2 * x) [4,5,6]
[8,10,12]
```

* **Sentido restrito**: programar **sem efeitos colaterais**, isto é, com **funções puras** — funções cujo resultado depende apenas dos argumentos, sem alterar estado externo nem depender dele. Uma função pura aplicada aos mesmos argumentos sempre devolve o mesmo resultado; essa propriedade é chamada de **transparência referencial**.

Mesmo em uma linguagem imperativa é possível — e desejável — escrever funções puras:

```c
double dobra(double x) {
    return 2 * x;
}
```

Mas nem toda função "parece" pura à primeira vista. Compare com a função abaixo — ela é pura?

```c
double i = 0;

double dobraMaisI(double x) {
    i += 1;              // efeito colateral: altera estado global
    return 2 * x + i;
}
```

Não: ela depende de e modifica uma variável global `i`, então duas chamadas com o mesmo `x` podem devolver resultados diferentes. Esse tipo de análise — "essa função é pura?" — é um exercício que vale a pena internalizar desde já, porque ele é a base de boa parte do raciocínio sobre corretude e testabilidade de código, em qualquer paradigma.

### Paradigma Lógico

Baseado em três conceitos centrais: **fatos**, **regras** e **perguntas**. Prolog é o representante clássico; você pode experimentar interativamente em [swish.swi-prolog.org](https://swish.swi-prolog.org/). Não é o foco deste curso, mas vale conhecer o modelo mental — programar *descrevendo relações*, e deixar que o motor de inferência descubra como satisfazê-las.

### Paradigma Orientado a Objetos

Esse é o paradigma que estudaremos em profundidade a partir da Unidade 2, usando Java como linguagem de estudo. Por ora, fica o resumo de Alan Kay (criador do Smalltalk, um dos berços da orientação a objetos): tudo é um objeto; objetos se comunicam trocando mensagens; cada objeto tem sua própria memória; todo objeto é instância de uma classe; a classe concentra o comportamento compartilhado por suas instâncias.

---

## ⚙️ Métodos de implementação

Além do paradigma, outra dimensão importante para comparar linguagens é **como** o código-fonte se transforma em execução:

| Método | Características | Exemplos |
|---|---|---|
| **Compilação** | Tradução direta para código de máquina antes da execução; foco em eficiência e sistemas | C, C++, Rust, Go |
| **Interpretação** | Código lido e executado passo a passo por outro programa, geralmente embutido em uma aplicação (navegador, shell, servidor) | Bash, linguagens de script |
| **Híbrido (máquina virtual)** | Compilação para um código intermediário (*bytecode*), executado depois por uma máquina virtual | Java (JVM), .NET (CLR), Erlang/Elixir (BEAM) |

A linguagem **Go** é um bom exemplo moderno de compilação cruzada eficiente: um único código-fonte compila nativamente para diversos sistemas operacionais e arquiteturas sem depender de máquina virtual.

Já a JVM se tornou um alvo de compilação popular para várias linguagens além do Java — Kotlin, Groovy, Scala e Clojure "compilam" para o mesmo bytecode; a BEAM (máquina virtual de Erlang) é o destino da linguagem brasileira **Elixir**.

Mesmo interpretadores usam representações intermediárias internamente. Em Python, por exemplo, é possível inspecionar o bytecode gerado para uma função:

```python
import dis

def dobro(x):
    return 2 * x

dis.dis(dobro)
```

Saída no Python 3.13 (os nomes das instruções mudam entre versões do interpretador):

```text
  3           RESUME                   0

  4           LOAD_CONST               1 (2)
              LOAD_FAST                0 (x)
              BINARY_OP                5 (*)
              RETURN_VALUE
```

---

## 📏 Características de linguagens de programação

Discussões sobre "qual linguagem é melhor" tendem a ser acaloradas e cheias de subjetividade. Sebesta propõe um conjunto de critérios mais objetivos para comparar linguagens — vamos usá-los ao longo de todo o curso.

### Legibilidade

Facilidade de **ler e entender** um programa já escrito.

**Simplicidade.** Uma linguagem com muitas construções básicas é mais difícil de aprender do que uma com poucas. Dois problemas comuns aqui:

* **Multiplicidade de recursos**: existir mais de uma forma de fazer a mesma coisa. Java, por exemplo, oferece quatro formas equivalentes de incrementar uma variável:

```java
count = count + 1;
count += 1;
count++;
++count;
```

* **Sobrecarga de símbolos**: usar o mesmo símbolo para significados diferentes conforme o contexto — em C, o que significam os três `*` a seguir?

```c
a = *p * *q;
```

Java optou por **não** oferecer sobrecarga de operadores (diferente de C++, que a tem), mas permite sobrecarga de **métodos**. Scala vai no sentido oposto e dá grande liberdade para definir operadores — poderosa, mas fácil de abusar. Go é ainda mais restritiva que Java e não oferece nenhuma das duas: seus projetistas argumentam que múltiplos métodos com o mesmo nome e assinaturas diferentes, embora ocasionalmente úteis, podem deixar o código confuso e frágil na prática.

**Ortogonalidade.** Diz respeito à capacidade de combinar os conceitos básicos da linguagem livremente, sem produzir efeitos anômalos — quanto menos exceções às regras gerais, mais ortogonal é a linguagem. Exemplos de baixa ortogonalidade:

* Em C, o tipo `void` só é válido em declarações de função e ponteiros — não existe "variável void".
* Em C, uma `struct` pode ser retornada por uma função, mas um array não pode.
* Em Java, por que a linha 4 abaixo não compila?

```java
int x, y = 2, z = 3;
byte a, b = 2, c = 3;
x = y + z;   // ok: int + int -> int
a = b + c;   // erro de compilação!
```

(A resposta tem a ver com **promoção de tipos**: em Java, uma expressão aritmética entre `byte`s é promovida a `int`, e o resultado `int` não pode ser atribuído a `byte` sem *cast* explícito — veremos isso em detalhe no capítulo sobre Tipos e Expressões.)

### Redigibilidade

A facilidade de **escrever** programas para um determinado domínio de problema. Note o "para um determinado domínio": R foi desenhada para facilitar análise de dados, não para escrever um sistema operacional — comparar redigibilidade fora do domínio-alvo é injusto com a linguagem.

Boa parte dos fatores que afetam redigibilidade também afeta legibilidade — abstração é o principal deles: funções, classes e outros mecanismos de abstração tornam o código mais curto e mais claro simultaneamente. **Expressividade** é outro fator: operadores poderosos que permitem expressar bastante computação em pouco código. Em C, o `for` não faz nada que o `while` não faça, mas deixa laços com contagem fixa mais curtos e mais fáceis de ler:

```c
int i = 0;
while (i < 10) {
    // comandos
    i++;
}

// o mesmo laço com for
for (int i = 0; i < 10; i++) {
    // comandos
}
```

Redigibilidade e legibilidade às vezes entram em conflito — código muito conciso pode ficar difícil de ler. O que faz o código C abaixo?

```c
void f(char *q, char *p) {
    for (; *q = *p; q++, p++);
}
```

(É uma cópia de string escrita de forma extremamente compacta — funcional, mas ilegível para quem não é fluente em C.)

### Confiabilidade

Um programa é confiável se ele se comporta conforme sua especificação **em todas as condições**. Elementos de linguagem que aumentam ou reduzem confiabilidade:

* **Checagem de tipos em tempo de compilação**: linguagens estaticamente tipadas capturam erros de tipo antes da execução — Java tende a ser mais confiável que Python nesse eixo.
* **Tipagem forte**: evita conversões implícitas perigosas. O código abaixo compila em C, mas não em C++ — porque C++ é mais fortemente tipada quanto a conversões implícitas de ponteiro:

```c
#include <stdlib.h>

void exemplo(void) {
    int *p = malloc(sizeof(int));  // void* → int*: aceito em C, erro em C++
    free(p);
}
```

* **Tratamento de exceções**: permite interceptar erros e, em certos casos, obriga o programador a tratá-los — em Java, exceções checadas (como as de acesso a banco de dados) precisam ser declaradas ou tratadas, algo que linguagens baseadas apenas em códigos de erro não conseguem impor.
* **Acesso direto a ponteiros**: manipular ponteiros livremente (como em C) é uma fonte clássica de bugs. C++ herda esse acesso irrestrito de C; Java e a maioria das linguagens mais recentes não expõem ponteiros ao programador.
* **Referências nulas**: mesmo sem ponteiros, o problema persiste — `NullPointerException` em Java é um dos erros mais comuns em produção. Linguagens mais recentes (Kotlin, Swift) tratam nulidade no próprio sistema de tipos; Java oferece `Optional<T>` desde a versão 8 como mecanismo opcional (não imposto pelo compilador), que veremos na Unidade 3.

Combinando esses fatores, uma ordem aproximada de confiabilidade seria: C < C++ < Java < Kotlin — cada uma reduzindo uma classe de erro que a anterior ainda permitia.

### Custos

Por fim, a escolha de uma linguagem para um projeto real envolve custos que vão muito além da sintaxe (Sebesta): custo de **treinar programadores**, de **escrever** programas, de **compilar**, de **executar**, de **manter** a implementação da própria linguagem, custo da **baixa confiabilidade** (bugs em produção) e custo de **manutenção** ao longo do tempo. Entre todos, os três mais decisivos na prática costumam ser desenvolvimento, manutenção e confiabilidade — não desempenho bruto.

---

## 🌍 Um panorama de linguagens do mercado

Não existe hoje um único projeto de software que use apenas uma linguagem. Conhecer, ainda que superficialmente, o "sabor" de várias linguagens ajuda a reconhecer padrões quando você encontrar uma nova. Um recorte razoável do mercado atual:

| Linguagem | Paradigma principal | Tipagem | Método de execução |
|---|---|---|---|
| Java | Orientado a objetos (com recursos funcionais desde a v8) | Estática, forte | Híbrido (JVM) |
| Python | Multiparadigma (imperativo, OO, funcional) | Dinâmica, forte | Interpretado / híbrido |
| JavaScript | Multiparadigma, baseado em protótipos | Dinâmica, fraca | Interpretado / JIT |
| Kotlin | Orientado a objetos + funcional | Estática, forte | Híbrido (JVM) |
| Go | Imperativo/estruturado, concorrente | Estática, forte | Compilado |
| Swift | Orientado a objetos + protocolos | Estática, forte | Compilado |
| C# | Orientado a objetos | Estática, forte | Híbrido (CLR) |
| Elixir | Funcional, orientado a atores | Dinâmica, forte | Híbrido (BEAM) |
| Clojure | Funcional (Lisp) | Dinâmica, forte | Híbrido (JVM) |

Ao pesquisar uma linguagem nova, é útil ter em mente um pequeno checklist de perguntas — as mesmas que usaremos para descrever Java na Unidade 1:

* O sistema de tipos é estático ou dinâmico? Há inferência de tipos?
* Qual é o paradigma predominante — e quais outros ela incorpora?
* É *class-based* ou *prototype-based*?
* Suporta casamento de padrões (*pattern matching*)?
* Funções são valores de primeira classe?

---

## 💡 Por que estudar linguagens de programação?

Segundo Sebesta, o estudo comparado de linguagens traz vantagens concretas, mesmo que você passe a carreira inteira programando em uma ou duas linguagens:

* Aumenta a capacidade de **expressar ideias** em código — você reconhece padrões de outras linguagens e os aplica na sua.
* Dá embasamento para **escolher a linguagem adequada** a cada problema, em vez de usar sempre o mesmo martelo.
* Aumenta a habilidade de **aprender novas linguagens** rapidamente — o vocabulário conceitual (paradigma, tipagem, escopo, ligação) se transfere.
* Melhora o uso de linguagens que você **já conhece**, ao entender *por que* elas tomaram certas decisões de design.
* Contribui para o **avanço geral da computação**, como profissional capaz de avaliar criticamente novas ferramentas.

---

## 📝 Atividades

1. Quais elementos da linguagem Python aumentam sua redigibilidade e legibilidade em comparação com C? Dê exemplos.
2. Por que, em geral, é possível escrever código mais eficiente em C do que em Python?
3. Reusabilidade e modificabilidade costumam se reforçar mutuamente. Dê um exemplo dessa relação.
4. Escolha três propriedades desejáveis de linguagens (por exemplo, legibilidade, confiabilidade, reuso) e, para cada uma, dê um exemplo concreto — de qualquer linguagem que você conheça — que ilustre como ela é alcançada.
5. Dê um exemplo, em qualquer linguagem, em que a busca por eficiência prejudica a confiabilidade.
6. Dê um exemplo em que a busca por portabilidade prejudica a eficiência.
7. Uma linguagem pode sempre ser implementada tanto por compilação quanto por interpretação? Pesquise linguagens que suportam ambos os métodos.
8. O que significa dizer que linguagens declarativas "não têm estado"? Compare com um exemplo imperativo equivalente.
9. Dê exemplos de conceitos originados em linguagens funcionais que hoje aparecem em linguagens mainstream mais antigas (incluindo o próprio Java).

---

## 🔗 Para saber mais

* Varejão, F. — [*Linguagens de Programação: Conceitos e Técnicas*](http://www.inf.ufes.br/~fvarejao/livroLP.html) (livro gratuito)
* [Timeline de linguagens de programação — Computer History Museum](https://www.computerhistory.org/timeline/software-languages/)
* [Timeline das linguagens de programação — Wikipedia](https://en.wikipedia.org/wiki/Timeline_of_programming_languages)
* Wegner, P. — *Concepts and Paradigms of Object-Oriented Programming* (palestra na OOPSLA 1989)
