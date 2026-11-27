# Lista de Exercícios

Uma lista de exercícios para revisar os conceitos discutidos ao longo do livro — organizados em dissertativas (cobrindo as três unidades) e uma seleção de questões de múltipla escolha do **POSCOMP** (Exame Nacional de Ingresso na Pós-Graduação em Computação), que costuma cobrar boa parte deste conteúdo.

## ✍️ Questões dissertativas

### Unidade 1 — Paradigmas e Fundamentos de Java

1. Qual a principal característica, vantagens e desvantagens das linguagens estaticamente tipadas? Exemplifique com código.
2. Qual a principal característica, vantagens e desvantagens das linguagens dinamicamente tipadas? Exemplifique com código.
3. Cite exemplos de linguagens estaticamente e dinamicamente tipadas.
4. O que é inferência de tipos? Exemplifique e cite linguagens que usam esse recurso — incluindo o próprio Java.
5. O que são paradigmas de programação? Cite os principais discutidos neste livro.
6. Quais conceitos surgiram com a programação estruturada?
7. Compare as linguagens Java e Python.
8. Compare as linguagens Java e C.
9. O que significa dizer que uma linguagem trata funções como valores de primeira classe?
10. Qual a diferença entre as estratégias de avaliação *call-by-value* e *call-by-reference*? Em qual delas se encaixa a passagem de parâmetros em Java?
11. Segundo Sebesta, quais critérios usamos para avaliar linguagens de programação? Escolha três e ilustre cada um com um exemplo.

### Unidade 2 — Orientação a Objetos

12. Defina e exemplifique objeto e classe.
13. O que é encapsulamento, e quais os seus benefícios? Exemplifique.
14. Qual a diferença entre atributos de instância e atributos estáticos (de classe)?
15. Qual a diferença entre métodos de instância e métodos estáticos?
16. Qual a diferença entre variáveis locais e atributos de instância?
17. O que é herança em orientação a objetos? Exemplifique.
18. Que problema(s) a herança tenta resolver?
19. Qual a diferença entre **sobrescrever** (*override*) e **sobrecarregar** (*overload*) um método?
20. Quais são os modificadores de controle de acesso em Java, e qual o efeito de cada um?
21. Qual a diferença entre classes abstratas e interfaces? Em que situação você escolheria uma ou outra?
22. Existe herança múltipla de classes em Java? E de interfaces?
23. Qual a diferença entre dependência e associação entre classes? Exemplifique.
24. Explique os quatro tipos de polimorfismo tratados neste livro (sobrecarga, coerção, paramétrico, inclusão).
25. Qual a diferença entre polimorfismo ad-hoc e polimorfismo universal?
26. Qual a diferença entre *cast* e coerção? Exemplifique.

### Unidade 3 — Java Avançado

27. O que é uma exceção, no contexto de Java?
28. O que significa "tratamento" ou "manipulação" de exceções?
29. Cite exemplos de classes de exceção do próprio Java, e classifique-as (checked/unchecked).
30. Explique o bloco `try`/`catch`/`finally`.
31. Explique a forma de lançamento de uma exceção (`throw new ...`).
32. Explique o funcionamento da propagação de exceções (`throws`).
33. O que é programação reflexiva?
34. No contexto de metaprogramação, o que é introspecção?
35. No contexto de metaprogramação, o que é uma anotação?
36. Quais as principais classes/APIs do Java que dão suporte à introspecção?
37. Por que dizemos que o Java dá amplo suporte à introspecção, mas apenas um suporte limitado à modificação por reflexão?
38. Cite exemplos de frameworks Java que fazem uso extensivo de metaprogramação.
39. O que são funções de alta ordem? Exemplifique com `map`, `filter` e `reduce` em Java.
40. Qual a diferença entre `Optional<T>` e simplesmente retornar `null`?

## 📝 Questões de múltipla escolha (POSCOMP)

As provas completas de anos anteriores estão disponíveis no [site oficial da SBC](https://www.sbc.org.br/documentos-da-sbc/category/153-provas-e-gabaritos-do-poscomp) — vale a pena resolver as provas completas, não só as questões de orientação a objetos. Uma pequena seleção para prática:

!!! question "POSCOMP 2018 — Questão 31"
    Assinale a alternativa correta em relação ao Paradigma Imperativo de Linguagens de Programação.

    A) Não é baseado na arquitetura de Von Neumann.
    B) É paradigma de linguagens não tipadas.
    C) É paradigma de linguagens orientadas a objeto (apenas).
    D) É baseado na arquitetura de Harvard.
    E) Descreve uma sequência de passos que mudam o estado de um programa.

    ??? success "Resposta"
        **E**. O paradigma imperativo é definido justamente pela mudança de estado através de comandos sequenciais (Capítulo 4) — e está historicamente ligado à arquitetura de Von Neumann (memória e programa compartilhando o mesmo espaço de endereçamento), não à de Harvard.

!!! question "POSCOMP 2018 — Questão 38"
    Sobre tipos de dados, é correto afirmar que:

    A) Tipos booleanos são valores que são mantidos fixos pelo compilador.
    B) O `double` é um tipo inteiro duplo, com menor precisão do que o tipo inteiro.
    C) A faixa de valores dos tipos inteiros tem somente dois elementos: um para verdadeiro e outro para falso.
    D) Uma conversão de tipos implícita consiste em uma modificação do tipo de dado, executada automaticamente pelo compilador.
    E) Vetores, matrizes e ponteiros são exemplos de tipos de dados primitivos (básicos).

    ??? success "Resposta"
        **D**. Isso é exatamente a definição de coerção/conversão implícita vista nos Capítulos 3 e 7 — realizada automaticamente pelo compilador, sem intervenção explícita do programador (como em `int + double`, ou na passagem de um `int` para um parâmetro `double`).

!!! question "POSCOMP 2018 — Questão 33"
    Quando um programa precisa classificar uma matriz de dados numéricos, normalmente usa um subprograma para o processo de classificação. No ponto em que a classificação é necessária, uma instrução como `sort_int(list, list_len)` é usada. Essa chamada é um exemplo de abstração de:

    A) Dados.
    B) Encapsulamento.
    C) Repetição.
    D) Condição.
    E) Processo.

    ??? success "Resposta"
        **E**. Chamar um subprograma sem se preocupar com os detalhes do algoritmo de ordenação usado é abstração de **processo** — o análogo, para comportamento, do que a abstração de dados (TAD, Capítulo 5) faz para estrutura de dados.

!!! question "POSCOMP 2017 — Questão 31"
    Quando uma expressão contém duas ocorrências adjacentes de operadores com o mesmo nível de precedência, a questão de qual é avaliado primeiro se resolve pelas regras de:

    A) Associatividade.
    B) Expressões condicionais.
    C) Polimorfismo.
    D) Condutividade.
    E) Coerção.

    ??? success "Resposta"
        **A**. Precedência decide entre operadores de **níveis diferentes** (`*` antes de `+`); associatividade decide entre operadores **adjacentes do mesmo nível** — vimos os dois conceitos aplicados a expressões Java no Capítulo 3.

!!! question "POSCOMP 2016 — Questão 28"
    Assinale a alternativa que apresenta o nome de uma linguagem de tipagem dinâmica.

    A) Java.
    B) C.
    C) Python.
    D) Pascal.
    E) C#.

    ??? success "Resposta"
        **C**. Java, C, Pascal e C# são todas estaticamente tipadas (Capítulo 3); Python é a única dinamicamente tipada da lista.

!!! question "POSCOMP 2018 — Questão 28"
    (Questão original envolve um diagrama de classes UML com as classes `Artefato`, `Veículo`, `Roda`, `Pneu` e `Aerofólio`, relacionadas por generalização e composição/agregação.) Assinale a alternativa correta sobre qual relacionamento do diagrama se conecta diretamente a polimorfismo.

    ??? tip "Como abordar"
        Esse tipo de questão exige olhar qual relação do diagrama é uma **generalização** ("é um") — só generalização (herança) habilita polimorfismo de inclusão (Capítulo 7); associações e composições ("tem um") não. Ao revisar provas completas do POSCOMP com diagramas, aplique esse filtro primeiro: identifique a seta de generalização antes de avaliar as alternativas.

## 📝 Exercício de síntese

Como fechamento do livro, escolha um problema do seu interesse (um pequeno sistema de agenda, biblioteca, loja, ou similar) e modele-o do zero, aplicando deliberadamente pelo menos:

* Uma hierarquia de herança com polimorfismo de inclusão (Capítulos 6 e 7).
* Uma interface com um método `default` (Capítulo 8).
* Um `record` para algum dado imutável (Capítulo 3/5).
* Tratamento de exceções customizadas para pelo menos uma regra de negócio (Capítulo 10).
* Uma coleção do Collections Framework, escolhida de forma justificada — não por padrão (Capítulo 11).

Esse exercício é, propositalmente, mais aberto que os anteriores — a ideia é exercitar as **decisões de design** discutidas ao longo do livro (Capítulo 9), não apenas a sintaxe de cada recurso isoladamente.

## 🔗 Para saber mais

* [Provas e gabaritos do POSCOMP — SBC](https://www.sbc.org.br/documentos-da-sbc/category/153-provas-e-gabaritos-do-poscomp)
* [SWI-Prolog online (SWISH)](https://swish.swi-prolog.org/) — para revisitar o paradigma lógico do Capítulo 1
