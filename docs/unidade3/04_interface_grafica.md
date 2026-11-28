# Cap 13: Interface Gráfica com Swing

!!! info "Por que Swing, e não uma biblioteca mais nova?"
    Esta parte da disciplina usa a biblioteca **Swing** apenas como veículo para destacar conceitos de orientação a objetos já vistos — herança, interfaces, o padrão Observer — não como um curso de desenvolvimento desktop. Vale registrar, de forma honesta: hoje, para aplicações desktop novas em Java, a alternativa mais moderna é o **JavaFX** (separado do JDK desde o Java 11, distribuído como módulo independente), com um modelo de layout mais flexível (CSS, FXML) e melhor suporte a gráficos modernos. O Swing continua relevante por dois motivos práticos: ainda sustenta uma quantidade enorme de software corporativo legado, e — para os fins pedagógicos deste capítulo — seu modelo de eventos é simples o suficiente para ilustrar bem o padrão Observer sem esconder os conceitos atrás de muita configuração.

## 🖼️ AWT e Swing

O **AWT** (*Abstract Window Toolkit*) foi o primeiro conjunto de ferramentas gráficas do Java — com componentes de aparência dependente de plataforma, recursos limitados (dependia do suporte de cada sistema operacional) e uma história conhecida de bugs de portabilidade. O **Swing** construiu sobre a base do AWT, mantendo compatibilidade, mas resolvendo os dois problemas centrais: um conjunto muito mais rico de componentes, e o conceito de **Look and Feel** — a aparência visual passa a ser decidida pela própria biblioteca Java, não pelo sistema operacional, eliminando as inconsistências entre plataformas.

## 🧱 Componentes e containers

Uma interface gráfica é composta de **componentes** (os elementos visíveis — botões, campos de texto, menus) organizados dentro de **containers**, que também podem aninhar outros containers.

```java
public class SwingOla {
    public static void main(String[] args) {
        JFrame frame = new JFrame();
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(200, 200);
        frame.setVisible(true);
    }
}
```

Como `JFrame` é um *container*, novos componentes podem ser adicionados a ele:

```java
JFrame frame = new JFrame();
JButton bt = new JButton("Diga Olá");
frame.add(bt);
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
frame.setSize(200, 200);
frame.setVisible(true);
```

### Layout

O Java precisa de uma política para organizar os componentes adicionados a um container. O padrão de um `JFrame` é o **BorderLayout** (já um `JPanel` usa, por padrão, o `FlowLayout`). O `BorderLayout` divide a janela em cinco regiões (norte, sul, leste, oeste, centro); adicionar um componente sem especificar região o coloca no centro:

```java
frame.add(bt, BorderLayout.CENTER); // equivalente a:
frame.add(bt);
```

O **FlowLayout** posiciona componentes da esquerda para a direita, quebrando linha quando necessário — como um editor de texto organizando palavras:

```java
JFrame frame = new JFrame();
frame.setLayout(new FlowLayout());
frame.setSize(250, 60);
frame.add(new JButton("Diga Olá"));
frame.add(new JButton("Diga Adeus"));
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
frame.setVisible(true);
```

O **GridLayout** divide o container em células invisíveis de tamanho igual:

```java
JFrame frame = new JFrame();
frame.setLayout(new GridLayout(2, 3));
frame.setSize(250, 150);
for (int i = 1; i < 7; i++) {
    frame.add(new JButton("Botão " + i));
}
frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
frame.setVisible(true);
```

Combinando `JPanel` (um container invisível) com layouts diferentes em cada painel, é possível compor layouts arbitrariamente complexos aninhando painéis.

## 🎯 Eventos: o padrão Observer em ação

O Swing organiza interação com o usuário em torno de **eventos**: objetos são divididos entre **fontes** (*sources*, como um `JButton`) e **receptores** (*listeners*), que reagem quando um evento — clique de mouse, tecla pressionada — acontece. Isso é, literalmente, o padrão de projeto **Observer** citado no Capítulo 9: a fonte notifica todos os seus *listeners* registrados quando seu estado muda (no caso, "foi clicado").

Para capturar o clique de um botão, é preciso um receptor que implemente a interface `ActionListener`:

```java
class Finaliza implements ActionListener {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.exit(0);
    }
}
```

```java
JButton btFechar = new JButton("Diga Adeus");
btFechar.addActionListener(new Finaliza());
```

Em vez de uma classe nomeada, uma **classe anônima** evita o boilerplate de declarar `Finaliza` separadamente:

```java
btFechar.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent ev) {
        System.exit(0);
    }
});
```

E como `ActionListener` tem um único método abstrato — é, na definição do Capítulo 8, uma **interface funcional** — desde o Java 8 é possível usar diretamente uma expressão lambda, muito mais concisa:

```java
btFechar.addActionListener(e -> System.exit(0));
```

### Organizando em uma classe própria

O padrão mais comum na prática é encapsular toda a montagem da interface dentro de uma subclasse de `JFrame`:

```java
public class PrimeiroSwing extends JFrame {
    public PrimeiroSwing() {
        setTitle("Primeiro Swing");

        JButton bt = new JButton("Diga Olá");
        bt.addActionListener(e ->
            JOptionPane.showMessageDialog(this, "Olá!!"));

        setLayout(new FlowLayout());
        setSize(250, 60);
        add(bt);

        JButton btFechar = new JButton("Diga Adeus");
        btFechar.addActionListener(e -> System.exit(0));
        add(btFechar);

        setDefaultCloseOperation(EXIT_ON_CLOSE);
    }
}
```

```java
public class SwingOla {
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            var janela = new PrimeiroSwing();
            janela.setVisible(true);
        });
    }
}
```

### A thread de eventos

O Swing não é *thread-safe*: os componentes devem ser criados e modificados na **thread de despacho de eventos** (*Event Dispatch Thread*, EDT), a mesma que executa os *listeners*. `SwingUtilities.invokeLater` agenda um trecho de código nessa thread, e por isso o `main` acima o usa. Os primeiros exemplos do capítulo omitem esse cuidado apenas para ficarem curtos.

A consequência inversa também importa: uma operação demorada dentro de um *listener* (uma consulta grande ao banco, por exemplo) **congela a interface** até terminar. Para esses casos, o Swing oferece `SwingWorker`, que executa o trabalho em outra thread e devolve o resultado à EDT.

## 📊 JTable: separando visualização de modelo de dados

`JTable` é o componente do Swing para apresentar dados em formato tabular — e ilustra bem outro princípio já visto (Capítulo 9): **separar a visualização do modelo**. A `JTable` não sabe *quais objetos* está mostrando; ela apenas consulta uma interface `TableModel`:

* `JTable` — cuida de visualização e interação, delegando os dados ao seu `TableModel`.
* `TableModel` — interface que define o que a `JTable` precisa para funcionar: `getValueAt()`, `setValueAt()`, `getRowCount()`, `getColumnCount()`.
* `AbstractTableModel` — classe abstrata (Capítulo 8) que já implementa parte do trabalho repetitivo (principalmente o disparo de eventos de mudança), deixando você sobrescrever só o que é específico do seu domínio.
* `TableModelListener` — o *listener* usado pela `AbstractTableModel` para notificar a `JTable` quando os dados mudam (Observer, de novo).
* `DefaultTableModel` — implementação concreta pronta, útil para dados estáticos:

```java
Object[][] dados = {
    {"Lucas", "Costa", "Natação", 5},
    {"Maria", "Silva", "Futebol", 3},
};
String[] colunas = {"Nome", "Sobrenome", "Esporte", "Tempo"};

var modelo = new DefaultTableModel(dados, colunas);
var tabela = new JTable(modelo);
```

Para dados vindos de uma fonte dinâmica — como o banco de dados do capítulo anterior — o caminho natural é escrever seu **próprio** `TableModel`, estendendo `AbstractTableModel` e delegando o carregamento a um DAO (padrão que veremos com mais detalhe no próximo capítulo, quando ele se junta a reflexão para eliminar boilerplate repetitivo).

Por padrão, as células de um `AbstractTableModel` **não são editáveis**. Para permitir a edição, sobrescreva `isCellEditable`, devolvendo `true`, e `setValueAt`, que guarda o novo valor e chama `fireTableCellUpdated` para avisar a `JTable`.

## 🧪 Laboratório

Continuando o projeto do capítulo anterior (JDBC): construa uma tela de cadastro (`JFrame` com campos de texto e um botão "Salvar", persistindo via JDBC) e uma tela de listagem usando `JTable`, alimentada pelos dados do banco. A tabela deve permitir editar registros existentes e refletir as mudanças no banco; opcionalmente, adicione um botão para excluir um registro selecionado.

## 📝 Atividades

1. Reimplemente o exemplo `PrimeiroSwing`, adicionando um `JTextField` e um botão que exibe, em um `JOptionPane`, o texto digitado. Identifique a fonte e o *listener* envolvidos.
2. Explique, em termos do padrão Observer (Capítulo 9), o papel de `ActionListener` e de `addActionListener`.
3. Pesquise a diferença entre AWT, Swing e JavaFX — em que cenário cada um ainda faz sentido hoje?
4. No exemplo de `JTable` com `DefaultTableModel`: o que aconteceria se os dados viessem de um banco que muda em tempo real (outro processo inserindo linhas)? Que papel a interface `TableModel` (em vez da classe concreta `DefaultTableModel`) desempenharia nesse cenário?

## 🔗 Para saber mais

* [Documentação oficial: Swing Tutorial (Oracle)](https://docs.oracle.com/javase/tutorial/uiswing/)
* [JavaFX — página oficial do projeto OpenJFX](https://openjfx.io/)
