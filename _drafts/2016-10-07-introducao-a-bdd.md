---
layout: post
title: "Introdução ao BDD"
description: "Tradução de 'Introducing BDD'"
category: development
tags: [development, agile]
---
{% include JB/setup %}

Esta é uma tradução do artigo
["Introducing BDD" de Dan North](https://dannorth.net/introducing-bdd/).

Eu tinha um problema. Enquanto estava usando e ensinando práticas ágeis, como o
test-driven development, ou desenvolvimento dirigido a testes (TDD) em projetos em
diferentes ambientes eu sempre passava pela mesma confusão e desentendimentos.
Programadores queriam saber onde começar, o que testar e o que não testar, quanto
testar de uma só vez, como chamar seus testes e como entender porque um teste falha.

Quanto mais fundo eu entrava no TDD, mais eu sentia que a minha própria jornada
tinha sido menos uma aprendizagem gradual do que uma série de becos sem saída.
Lembro-me de pensar "Se alguém tivesse me dito isso!" muito mais vezes do que eu
pensava "Uau, uma porta se abriu". Eu imaginei que devia ser possível apresentar o TDD
de uma maneira que iria direto para as coisas boas e evitasse todas as armadilhas.

Minha resposta é o Behaviour-Driven Development, ou Desenvolvimento Dirigido a
Comportamento (BDD). Ele tem evoluido de práticas ágeis já estabelecidas e é projetado
para tornar essas práticas mais acessíveis e efetivas para equipes novas à
entrega ágil de software. Com o passar do tempo, o BDD cresceu para abranger
o panorama mais amplo da análise ágil e teste de aceitação automatizado.

## Nomes de métodos de teste devem ser frases

My first “Aha!” moment occurred as I was being shown a deceptively simple utility called agiledox, written by my colleague, Chris Stevenson. It takes a JUnit test class and prints out the method names as plain sentences, so a test case that looks like this:

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

Desenvolvedores descobriram que dessa maneira eles poderiam fazer ao menos
alguma documentação, então, eles começaram a escrever métodos de teste cujos nomes
eram sentenças reais. Além do mais, eles descobriram que quando eles escreviam os
nomes dos métodos na linguagem do domínio do negócio, os documentos gerados faziam
sentido para os usuários, analistas e testadores.

## Um modelo simples de frase mantém métodos de teste focados

Então me deparei com a convenção de começar nomes de métodos de teste com a palavra
"should", ou "deve", em português. "A classe deve fazer algo", significa que você
só pode definir um teste para a classe atual. Isso mantém você focado. Se você
se depara escrevendo um teste cujo nome não se encaixa neste modelo, ele sugere
que o comportamento pode pertencer a outro lugar.

Por exemplo, eu estava escrevendo uma classe que validava a entrada de uma tela.
A maioria dos campos eram detalhes normais de clientes - nome próprio, apelido e etc -
mas, em seguida há um campo para a data de nascimento e um para a idade. Eu comecei
escrevendo um `ValidadorDetalhesClienteTest` com métodos como
`testDeveFalharParaApelidoFaltando` e `testDeveFalharParaTituloFaltando`

Então eu comecei a calcular a idade e entrou num mundo de regras de negócios
complicadas: E se a idade e data de nascimento são ambas fornecidas, mas não
coincidem? E se o aniversário é hoje? Como faço para calcular a idade, se eu só
tenho uma data de nascimento? Eu estava escrevendo nomes de métodos de teste cada
vez mais complicados para descrever este comportamento, então eu considerei largar
essa abordagem para testar algo diferente. Isso me levou a introduzir uma nova
classe que eu chamei de `CalculadoraIdade`, com sua própria `CalculadoraIdadeTest`.
Todo o comportamento envolvido no cálculo de idade mudou-se para a calculadora,
então, o validador precisava apenas de um teste em torno do cálculo da idade para
garantir que ele interagiu adequadamente com a calculadora.

Se uma classe está fazendo mais de uma coisa, eu normalmente tomo isso como uma
indicação de que eu deveria introduzir outras classes para fazer parte do trabalho.
Eu defino o novo serviço como uma interface descrevendo o que ele faz, e eu passo
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

Depois de um tempo, eu descobri que se eu mudasse um código e isso causasse um teste
falhar, eu poderia olhar o nome do método de teste e identificar o comportamento
desejado do código. Tipicamente, uma das três coisas tinham acontecido:

- Eu tinha introduzido um bug. Culpa minha. Solução: Corrigir o bug
- O comportamento desejado ainda era relevante, mas tinha-se mudado para outro
lugar. Solução: Mover o teste e talvez alterá-lo
- O comportamento não estava mais correto - a premissa do sistema havia mudado.
Solução: Excluir o teste.

O último é provável que ocorra em projetos ágeis a medida que entendimento evolua.
Infelizmente, TDDers - praticantes de TDD - novatos tem um medo inato de excluir
testes, como se de alguma maneira isso reduzisse a qualidade de seu código.

Um aspecto mais sutil da palavra "should" ("deve" em português) se torna aparente
quando comparada com as alternativas mais formais, como "will" ("irá" em português),
ou "shall" ("deverá" em português). "Should" implicitamente te permite desafiar a
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
TDD e descobri que não somente parecia se encaixar mas também que toda uma
categoria de dúvidas a respeito do TDD haviam magicamente sido respondidas.
Agora eu tinha respostas a algumas daquelas questões a respeito do TDD. Como chamar
seu teste é fácil - é uma sentença que descreva o próximo comportamento no qual
você está interessado. Quanto testar de uma só vez se torna discutível - é o tanto
que você consegue descrever em uma única frase. Quando um teste falha, basta trabalhar
no processo descrito acima - se você introduziu um bug, o comportamento mudou de
lugar ou se o teste não é mais relevante.

Eu descobri a mudança de pensar em testes pensando em comportamento, de uma maneira
tão profunda que eu comecei a me referir ao TDD como BDD, ou Behaviour-Driven
Development.

## O JBehave enfatiza comportamento ao longo do teste

No final de 2003, eu decidi que era hora de colocar meu dinheiro - ou ao menos
meu tempo - no que eu acreditava. Eu comecei a desenvolver um substituto para o
JUnit, chamado JBehave, que removia qualquer referência a "teste" e a substituia
por um vocabulário constuído ao redor de verificar comportamento. Eu fiz isso para
ver quanto um framework evoluiría se eu aderisse estritamente aos meus novos
mantras dirigidos a comportamento. Eu também pensei que também seria uma ferramenta
valiosa de ensino para introduzir o TDD e BDD sem as distrações do vocabulário
baseado em teste.

Para definir o comportamento para uma classe hipotética `PesquisarCliente`, eu
criaria uma classe de comportamento chamada, por exemplo, `PesquisarClienteBehaviour`.
Ela conteria métodos que iniciariam com a palavra "should" (ou "deve").
O runner (classe que executa a classe de teste) do comportamento instanciaria a
classe de comportamento e invocaria cada um dos métodos de comportamento, por sua
vez, como o JUnit faz com seus testes. Ele iria reportar progresso a medida que
avança e imprimir um resumo ao final.

Meu marco inicial foi fazer que o JBehave se auto-verificasse. Eu somente adicionei
comportamento que o permitia executar por conta própria. Eu era capaz de migrar todos
os testes JUnit para os comportamentos JBehave  e obter o mesmo retorno imediato
que obteria com o JUnit.
