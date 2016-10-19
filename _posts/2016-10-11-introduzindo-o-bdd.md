---
layout: post
title: "Introduzindo o BDD"
description: "Tradução de 'Introducing BDD'"
category: development
tags: [development, agile]
---
{% include JB/setup %}

Esta é uma tradução do artigo
["Introducing BDD" de Dan North](https://dannorth.net/introducing-bdd/).

*História: Este artigo apareceu primeiro na revista
[Better Software](http://j.mp/vHRQWp) em Março de 2006. Ele foi traduzido
para o [Japonês](http://j.mp/vJOLGl) por [Yukei Wachi](http://j.mp/uIwtTO),
[Coreano](http://j.mp/tIqbyZ) por [HongJoo Lee](http://j.mp/tr2Z5o),
[Italiano](http://bit.ly/v2twov) por [Arialdo Martini](http://bit.ly/vkjZBB),
[Francês](http://j.mp/wPipHF) por [Philippe Poumaroux](http://j.mp/zKyKgT),
[Espanhol](http://j.mp/157tvAS) por [Oscar Zárate](https://twitter.com/OZcarZarate),
[Turco](http://j.mp/1d2jpZa) por [Selim Öber](https://twitter.com/selimober),
[Russo](http://habrahabr.ru/post/216923/) por alguém chamado
[@w1ld](https://habrahabr.ru/users/w1ld/), [Persa](http://softengine.blog.ir/1395/04/21/introduction-to-bdd-dan-north) por
[Abouzar Kamaee](https://ir.linkedin.com/in/abouzarkamaee) e
[Alemão](https://www.yulup.com/de/lektuere/dan_north_einfuehrung_in_bdd.html) por
[Ivan Kellenberger](https://www.linkedin.com/in/ivankellenberger)*.

Eu tinha um problema. Enquanto estava usando e ensinando práticas ágeis, como o
Test-Driven Development, ou Desenvolvimento Dirigido a Testes (TDD) em projetos em
diferentes ambientes, eu sempre passava pela mesma confusão e desentendimentos.
Programadores queriam saber por onde começar, o que testar e o que não testar, quanto
testar de uma só vez, como chamar seus testes e como entender porque um teste falha.

Quanto mais fundo eu entrava no TDD, mais eu sentia que a minha própria jornada
tinha sido menos uma aprendizagem gradual do que uma série de becos sem saída.
Lembro-me de pensar "Se alguém tivesse me dito isso!" muito mais vezes do que eu
pensava "Uau, uma porta se abriu". Eu imaginava ser possível apresentar
o TDD de uma maneira que fosse direto para as coisas boas e evitasse todas as
armadilhas.

Minha resposta é o Behaviour-Driven Development, ou Desenvolvimento Dirigido a
Comportamento - BDD. Ele tem evoluido de práticas ágeis já estabelecidas e é projetado
para tornar essas práticas mais acessíveis e efetivas para equipes novas à
entrega ágil de software. Com o passar do tempo, o BDD cresceu para abranger
o panorama mais amplo da análise ágil e teste de aceitação automatizado.

## Nomes de métodos de teste devem ser frases

Meu primeiro "Aha!" ocorreu enquanto eu estava mostrando um utilitário extremamente
simples chamado [agiledox](http://agiledox.sourceforge.net/), escrito pelo meu colega,
Chris Stevenson. Ele pega uma classe de teste JUnit e imprime os nomes dos métodos
como frases simples, portanto, um caso de teste como esse:

```
public class PesquisarClienteTest extends TestCase {
  testBuscarClientePorId() {
    ...
  }

  testFalharPorClientesDuplicados() {
    ...
  }

  ...
}
```

Renderiza algo como isso:

```
PesquisarCliente
- buscar cliente por id
- falhar por clientes duplicados
- ...
```

A palavra "test" é removida tanto do nome da classe quanto dos nomes dos métodos,
e a notação camel-case é convertida para texto normal. Isso é tudo o que o programa
faz, mas seu efeito é incrível.

Desenvolvedores descobriram que, dessa maneira, eles poderiam fazer ao menos
alguma documentação, então, começaram a escrever métodos de teste cujos nomes
eram sentenças reais. Além do mais, descobriram que quando eles escreviam os
nomes dos métodos na linguagem do domínio do negócio, os documentos gerados faziam
sentido para os usuários, analistas e testadores.

## Um modelo simples de frase mantém métodos de teste focados

Então me deparei com a convenção de começar nomes de métodos de teste com a palavra
"should" ("deve", em português). "A classe deve fazer algo", significa que você
só pode definir um teste para a classe atual. Isso mantém você focado. Se você
se depara escrevendo um teste cujo nome não se encaixa neste modelo, ele sugere
que o comportamento pode pertencer a outro lugar.

Por exemplo, eu estava escrevendo uma classe que validava a entrada de uma tela.
A maioria dos campos eram detalhes normais de clientes - nome próprio, apelido e etc -
mas, em seguida há um campo para a data de nascimento e um para a idade. Eu comecei
escrevendo um `ValidadorDetalhesClienteTest` com métodos como
`testDeveFalharParaApelidoFaltando` e `testDeveFalharParaTituloFaltando`

Então eu comecei a calcular a idade e entrou num mundo de regras de negócios
complicadas: E se a idade e data de nascimento forem ambas fornecidas, mas não
coincidem? E se o aniversário é hoje? Como faço para calcular a idade, se eu só
tenho uma data de nascimento? Eu estava escrevendo nomes de métodos de teste cada
vez mais complicados para descrever este comportamento, então eu considerei largar
essa abordagem para testar algo diferente. Isso me levou a introduzir uma nova
classe que eu chamei de `CalculadoraIdade`, com sua própria `CalculadoraIdadeTest`.
Todo o comportamento envolvido no cálculo de idade mudou-se para a calculadora,
então, o validador precisava apenas de um teste em torno do cálculo da idade para
garantir que ele interagiu adequadamente com a calculadora.

Se uma classe está fazendo mais de uma coisa, eu normalmente tomo isso como uma
indicação de que mais classes deveriam ser introduzidas para fazerem parte do trabalho.
Eu defino o novo serviço como uma interface descrevendo o que ele faz, e passo
este serviço através de construtor da classe:

```
public class ValidadorDetalhesCliente {

  private final CalculadoraIdade calcIdade;

  public ValidadorDetalhesCliente(CalculadoraIdade calcIdade) {
    this.calcIdade = calcIdade;
  }
}
```

Esse estilo de conectar objetos, conhecido como injeção de dependência, é
especialmente útil em conjunção com mocks.

## Um nome de teste expressivo é útil quando um teste falha

Depois de um tempo, descobri que se eu mudasse um código e isso causasse um teste
falhar, eu poderia olhar para o nome do método de teste e identificar o comportamento
desejado do código. Tipicamente, uma das três coisas tinham acontecido:

- Eu tinha introduzido um bug. Culpa minha. Solução: Corrigir o bug
- O comportamento desejado ainda era relevante, mas tinha-se mudado para outro
lugar. Solução: Mover o teste e talvez alterá-lo
- O comportamento não estava mais correto - a premissa do sistema havia mudado.
Solução: Excluir o teste.

O último é provável que ocorra em projetos ágeis a medida que o entendimento evolua.
Infelizmente, TDDers - praticantes de TDD - novatos tem um medo inato de excluir
testes, como se de alguma maneira isso reduzisse a qualidade de seu código.

Um aspecto mais sutil da palavra "should" se torna aparente quando comparada com
as alternativas mais formais, como "will" e "shall" - respectivamente "irá" e
"deverá", em português. "Should", implicitamente, te permite desafiar a
premissa do teste: "Deveria? Mesmo?". Isso torna mais fácil decidir se um teste
está falhando devido a um bug que você introduziu ou simplesmente porque suas
suposições anteriores sobre o comportamento do sistema agora estão incorretas.

## "Comportamento" é uma palavra mais útil que "teste"

Agora eu tinha uma ferramenta - o [agiledox](http://agiledox.sourceforge.net/) -
que removia a palavra "test" e um modelo para cada método de teste. De repente
me ocorreu que os mal-entendidos das pessoas a respeito do TDD são quase sempre
sobre a palavra "teste".

Isso não é dizer que o teste não é intrínseco ao TDD - o conjunto resultante de
métodos é uma maneira efetiva de garantir que seu código funciona. Contudo, se os
métodos não descreverem completamente o comportamento do seu sistema, eles estarão
te causando uma falsa sensação de segurança.

Eu comecei a usar a palavra "comportamento" no lugar de "teste" quando usava o
TDD e descobri que não somente parecia se encaixar, mas também que toda uma
categoria de dúvidas a respeito do TDD havia sido magicamente dissolvida.
Agora eu tinha respostas a algumas daquelas questões a respeito do TDD. Como chamar
seu teste é fácil - é uma sentença que descreva o próximo comportamento no qual
você está interessado. Quanto testar de uma só vez se torna discutível - é o tanto
que você consegue descrever em uma única sentença. Quando um teste falha, basta
trabalhar no processo descrito acima - se você introduziu um bug, o comportamento
mudou de lugar ou se o teste não é mais relevante.

Eu descobri a mudança, de pensar em testes para pensar em comportamento, de uma
maneira tão profunda que eu comecei a me referir ao TDD como BDD, ou,
Behaviour-Driven Development.

## O JBehave enfatiza comportamento ao longo do teste

No final de 2003, eu decidi que era hora de colocar meu dinheiro - ou ao menos
meu tempo - no que eu acreditava. Eu comecei a desenvolver um substituto para o
JUnit, chamado JBehave, que removia quaisquer referências a teste e as substituia
por um vocabulário baseado na verificação de comportamento. Eu fiz isso para
ver quanto um framework evoluiría se eu aderisse estritamente aos meus novos
mantras dirigidos a comportamento. Eu também pensei que também seria uma ferramenta
valiosa de ensino para introduzir o TDD e BDD, sem as distrações do vocabulário
baseado em teste.

Para definir o comportamento para uma classe hipotética `PesquisarCliente`, eu
criaria uma classe de comportamento chamada, por exemplo, `PesquisarClienteBehaviour`.
Ela conteria métodos que iniciariam com a palavra "deve". O runner (classe que
executa a classe de teste) do comportamento, instanciaria a
classe de comportamento e invocaria cada um dos métodos de comportamento, por sua
vez, como o JUnit faz com seus testes. Ele iria reportar progresso a medida que
avança e imprimir um resumo ao final.

Meu marco inicial foi fazer com que o JBehave se auto-verificasse. Eu somente adicionei
comportamento que o permitia executar por conta própria. Eu era capaz de migrar todos
os testes JUnit para os comportamentos JBehave  e obter o mesmo retorno imediato
que obteria com o JUnit.

## Determine o próximo comportamento mais importante

Então eu descobri o conceito de valor de negócio. É claro, eu sempre estive
ciente de que eu desenvolvia software por uma razão, mas eu nunca realmente havia
pensado no valor do código que eu estava escrevendo no momento. Outro colega,
o analista de negócios Chris Matts, me fez pensar sobre o valor do negócio no
contexto do Behaviour-Driven Development.

Dado que eu tinha a meta em mente - de fazer JBehave em auto-hospedagem ("self-hosting",
em inglês) - descobri que uma maneira muito útil para se manter focado era perguntar:
Qual é a próxima coisa mais importante que o sistema ainda não faz?

Essa questão requer que você identifique o valor das funcionalidades que ainda não
implementou e priorizá-las. Isso ainda te ajuda a formular o nome do comportamento:
O sistema não faz X (onde X é algum comportamento significativo), e X é importante,
o que significa que deveria fazer X; portanto, o próximo método de comportamento
é simplesmente:

```
public void deveFazerX() {
    // ...
}
```

## Requisitos são comportamento, também

Neste ponto, eu tinha um framework que me ajudou a entender - e mais importante,
a explicar - como o TDD funciona e uma abordagem que evitou todas as armadilhas
que eu tinha encontrado.

Para o fim de 2004, enquanto eu estava descrevendo meu novo vocabulário encontrado
baseado em comportamento para Matts, ele disse: "Mas isso é apenas como a análise".
Houve uma longa pausa enquanto nós processávamos isto, e em seguida, decidimos
aplicar todo este pensamento dirigido a comportamento na definição de requisitos.
Se pudéssemos desenvolver um vocabulário consistente para analistas, testadores,
desenvolvedores e pessoas da área de negócios, então estaríamos a caminho de
eliminar algumas das ambiguidades e falhas de comunicação que ocorrem quando
pessoas da área de tecnologia falam com pessoas da área de negócios.

## BDD fornece uma "linguagem ubíqua" para análise

Mais ou menos na mesma época, Eric Evans publicou seu livro best-seller "Domain Driven
Project". Nele, ele descreve o conceito de modelagem de um sistema utilizando uma
linguagem ubíqua baseada no domínio de negócio, de modo que o vocabulário de negócios
esteja no meio do código da aplicação.

Chris e eu percebemos que estávamos tentando definir uma linguagem ubíqua para o
próprio processo de análise! Tivemos um bom ponto de partida. No uso comum,
dentro da empresa, já havia um modelo de história que parecia assim:

```
Como um [X]
Eu quero [Y]
De modo a [Z]
```

Onde Y é uma funcionalidade, Z é o benefício ou valor da funcionalidade e X é
a pessoa (ou papel) que irá se beneficiar da funcionalidade. Seu ponto forte é
que ele te obriga a identificar o valor de entregar uma história quando você
a define. Quando não há real valor de negócio para a história, frequentemente
se resume a algo como "... Eu quero [alguma funcionalidade] de modo que
[eu apenas quero, OK?]". Isso torna mais fácil para reduzir o escopo de alguns
requisitos mais esotéricos.

A partir deste ponto de partida, Matts e eu começamos a descobrir o que cada tester
ágil já sabe: O comportamento de uma história é simplesmente seus critérios de
aceitação - se o sistema cumpre todos os critérios de aceitação, ele está se
comportando corretamente; caso contrário, ele não está se comportando corretamente.
Por isso, criamos um modelo para capturar os critérios de aceitação de uma história.

O modelo tinha que ser "relaxado" o suficiente de modo a não se sentir artificial
ou restringente para os analistas, mas estruturado o suficiente para que seja
possível quebrar a história em seus fragmentos constituintes e automatizá-los.
Começamos descrevendo os critérios de aceitação em termos de cenários, que teve
a seguinte forma:

```
Dado algum contexto inicial ("givens", em inglês),
Quando um evento ocorre,
então, garanta alguns resultados.
```

Para ilustrar, vamos usar o exemplo clássico da máquina de caixa eletrônico.
Um dos cartões de história se parece desse jeito:

```
+Título: Cliente saca dinheiro+

Como um cliente,
Eu quero sacar dinheiro do caixa eletrônico,
de modo que eu não tenha que esperar na fila do banco.
```

Então, como vamos saber quando nós entregamos essa história? Há vários cenários a se
considerar: a conta pode ter crédito, a conta pode pode estar no negativo,
mas dentro do limite de cheque especial, a conta pode estar no negativo além do
limite de cheque especial. É claro, haverá outros cenários, como se a conta está
com saldo positivo mas a retirada faz com que fique com saldo negativo, ou se
o dispensador tem dinheiro insuficiente.

Usando o modelo "dado-quando-então" ("given-when-then", em inglês), os dois
primeiros cenários podem se parecer dessa maneira:

```
+Cenário 1: A conta está com saldo positivo+

Dada a conta que está com saldo positivo,
E o cartão seja válido,
E o dispensador contenha dinheiro,
Quando o cliente requisita dinheiro
Então, garanta que a conta seja debitada,
E garanta que o dinheiro seja dispensado,
E assegure-se que o cartão seja devolvido,
```

Observe o uso do "e" para conectar vários "givens" ou múltiplos resultados de uma
forma natural.

```
+Cenário 2: A conta está com saldo negativo além do limite de cheque especial+

Dada a conta com saldo negativo,
E o cartão seja válido
Quando o cliente solicita dinheiro
Então, garanta que uma mensagem de rejeição seja exibida,
E garanta que dinheiro não seja dispensado,
E assegure-se de que o cartão seja devolvido.
```

Ambos os cenários são baseados no mesmo evento e até mesmo tem alguns "givens"
e resultados em comum. Queremos aproveitar isso através da reutilização de "givens",
eventos e saídas.

## Critérios de aceitação devem ser executáveis

Os fragmentos do cenário - os "givens", eventos e resultados - são de
baixa-granularidade o suficiente para serem representados diretamente em código.
O JBehave define um modelo de objeto que nos permite mapear diretamente os
fragmentos do cenário às classes Java.

Você escreve uma classe representando cada "given":

```
public class ContaEstaPositiva implements Given {
    public void setup(World world) {
        ...
    }
}

public class CartaoEValido implements Given {
    public void setup(World world) {
        ...
    }
}
```

e uma para o evento:

```
public class ClienteSolicitaDinheiro implements Event {
    public void occurIn(World world) {
        ...
    }
}
```

e assim por diante para os resultados. O JBehave, em seguida, conecta tudo isso
e os executa. Ele cria um "mundo" ("world", em inglês), que é apenas um lugar
para armazenar seus objetos, e passa para cada um dos "givens", para que eles
possam preencher o "mundo" com o estado conhecido. O JBehave, então, diz que o
evento ocorre no mundo (ou "occur in", em inglês), onde de fato executa o
comportamento real do cenário. Finalmente, ele passa o controle para quaisquer
resultados que definimos para a história.

Ter uma classe para representar cada fragmento nos permite reutilizá-los
em outros cenários ou histórias. No início, os fragmentos são implementados usando
mocks (ou "simulações" em português) para definir uma conta com saldo positivo e
cartão válido. Estes formam o ponto de partida para implementar o comportamento.
À medida que você implementa a aplicação, os "givens" e resultados são alterados
para utilizarem as classes reais que você implementou, de modo que, quando o cenário
está concluído, ele se torna um conjunto de testes funcionais de ponta-a-ponta,
ou end-to-end em inglês.

## O presente e futuro do BDD

Após um breve hiato, o JBehave está sob desenvolvimento ativo. Seu núcleo está
bastante completo e robusto. A próxima etapa é a integração com IDEs Java, como
o IntelliJ IDEA e o Eclipse.

[Dave Astels](http://daveastels.com/) tem promovido ativamente o BDD. Seu blog
e vários artigos publicados tem provocado agitação, mais notavelmente o projeto
[rspec](http://rspec.rubyforge.org/) - framework BDD desenvolvido em Ruby.
Eu comecei a trabalhar no rbehave, que será a implementação do JBehave em Ruby.

Alguns de meus colegas tem usado técnicas BDD em uma variedade de projetos
e acharam as técnicas bem sucedidas. O runner de histórias do JBehave - parte
que verifica os critérios de aceitação - está em desenvolvimento ativo.

A visão é ter um editor que permita aos analistas de negócio e testers
capturarem stubs para as classes de comportamento, tudo na linguagem do domínio
do negócio. O BDD evoluiu com a ajuda de muitas pessoas e eu sou imensamente
grato por todas elas.
