---
layout: post
title:  "How I use TDD in practice"
date:   2020-08-02 14:30:41 -0300
categories: tools
---
O que irei mostrar é uma introdução na prática de como programar usando a técnica do TDD e que posteriormente poderá ser aplicado em sua rotina de programação. Ainda sim, recomendo se aprofundar com algumas referências que deixarei ao final deste post.

É importante salientar que o TDD nos força a pensar no que realmente queremos como produto final antes mesmo de codificar e - o melhor de tudo - com os testes prontos, criados em um ciclo de teste, implementação e refatoração. Isso nos poupa muitas voltas implementando coisas desnecessárias ou até mesmo refatorando muitas vezes só por não termos pensado um pouco melhor no início, evita bugs, promove um código mais simples e feedbacks rápidos. Resumindo, o objetivo é ser produtivo $$$.

Sem mais delongas, bora praticar!

## Desenvolvendo com TDD

![TDD Cicle image](/assets/images/tdd-cicle.png "TDD Cicle")
*<center>Image from: https://marsner.com/blog/why-test-driven-development-tdd</center>*

#### Requisitos

A nossa feature (fictícia) será um sistema de pontuação.

#### Ferramentas

- Escolhi C++ para este exemplo, mas os princípios aplicados serão os mesmos em qualquer linguagem e/ou paradigma de programação.
- Vamos usar também a framework de teste chamada catch.
- Devemos anotar os pontos a serem desenvolvidos e cobertos pelos testes, portanto, não pode faltar um checklist (que pode ser feito com papel e lápis, o que eu prefiro).

Vamos começar com:

**Passo 1:** Criação do checklist da implementação dos requisitos técnicos, podendo ser de baixo ou alto nível.

- **sum score**
- subtract score

Neste passo, definimos o checklist e destacamos o que iremos trabalhar inicialmente, neste caso, a função de soma de pontuação.

**Passo 2:** Criar o teste para a implementação da soma.

testScore.cpp

{% highlight c++ %}
TEST_CASE("Score") {
    SECTION("test sum") {
        Score* score = new Score();
        score->sum(45);
        CHECK(score->value == 45);
    }
}
{% endhighlight %}

**Passo 3:** Execute os testes e veja-o falhar.

Neste passo você irá perceber que o testes serão nosso guia pra as implementações, em pequenos passos e com grande satisfação de um código funcional e testado.

**Passo 4:** Implemente a classe Score e sua função sum para fazer o teste passar.

Score.hpp

{% highlight c++ %}
class Score
{
public:
    int value = 0;
    void sum(int value);
};
{%endhighlight %}

Score.cpp

{% highlight c++ %}
void Score::sum(int value)
{
    this->value = 45;
};
{%endhighlight %}

**Passo 5:** Execute os testes novamente e veja a barra verde de sucesso.

A implementação duplicada foi proposital, pois será eliminada no próximo passo.

**Passo 6:** Refatore para eliminar duplicidade

Score.cpp

{% highlight c++ %}
void Score::sum(int value)
{
    this->value += value;
};
{%endhighlight %}

Podemos adicionar mais alguns testes para garantir a soma correta.

testScore.cpp

{% highlight c++ %}
TEST_CASE("Score") {
    SECTION("test sum") {
        Score* score = new Score();
        score->sum(45);
        CHECK(score->value == 45);
        score->sum(10);
        CHECK(score->value == 55);
    }
}
{% endhighlight %}

Maravilha! Finalizamos o ciclo da implementação de um dos requisitos.

Agora vamos reiniciar os passos para a implementação do próximo. 

Entendemos que um sistema de pontuação também precisa ter uma forma de comparar um ponto com o outro. Pensando nisso, vamos incrementar nosso checklist com mais duas coisas: comparador de igualdade e tornar “value” privado. “Tornar value privado” diz respeito a visibilidade que, por segurança, decidimos privar os usuários de alterem diretamente essa propriedade.

Portanto:

**Passo 1:** Atualizar o checklist

- subtract score
- **Make “value” private**
- equal
- ~~sum score~~

**Passo 2:** Alterar o teste para tornar “value” privado

{% highlight c++ %}
TEST_CASE("Score") {
    SECTION("test sum") {
        Score score(10);
        score.sum(45);
        Score toCompare(55);
        CHECK(score == toCompare);
        score.sum(10);
        Score toCompare2(65);
        CHECK(score == toCompare2);
    }
}
{% endhighlight %}

**Passo 3:** Execute o teste e veja-o falhar

**Passo 4:** Faça o teste passar.

Score.hpp

{% highlight c++ %}
class Score
{
private:
    int value = 0;
public:
    Score(int value);
    void sum(int value);
    bool operator==(const Score& s) const
    {
        return true;
    };
};
{%endhighlight %}

Score.cpp

{% highlight c++ %}
Score::Score(int value)
{
    this->value = value;
};

void Score::sum(int value)
{
    this->value += value;
};
{%endhighlight %}

**Passo 5:** Refatore para eliminar duplicidade

Score.hpp

{% highlight c++ %}
class Score
{
private:
    int value = 0;
public:
    Score(int value);
    void sum(int value);
    bool operator==(const Score& s) const
    {
        return this->value == s.value;
    };
};
{%endhighlight %}

Ótimo! Encerramos mais um ciclo. 

Olhando de novo para o código percebemos que há um risco de ocorrer um efeito colateral, pois a pontuação pode somar alterando a sua referência em vez de criar uma nova instância, portanto atualizaremos novamente nosso checklist e então trabalharemos nisso o mais rápido possível e já evitar dores de cabeça no futuro.

**Passo 1:** Atualizar o checklist

- subtract score
- equal
- **side-effects**
- ~~Make “value” private~~
- ~~sum score~~

**Passo 2:** Altere o teste para garantir que não há side-effects

{% highlight c++ %}
TEST_CASE("Score") {
    SECTION("test sum") {
        Score score(10);
        Score newScore = score.sum(45);
        Score toCompare(55);
        CHECK(newScore == toCompare);
        Score toCompareWithInitial(10);
        CHECK(score == toCompareWithInitial);
    }
}
{% endhighlight %}

Eliminamos o segundo assert do teste, pois entendemos que apenas um é suficiente para validar a nova instância de Score criada a partir do método “sum”. Sinta-se a vontade para criar e remover código sempre que necessário. Isso não é retrabalho. Pelo contrário!

**Passo 3:** Execute o teste e veja-o falhar

**Passo 4:** Faça o teste passar.

Score.hpp

{% highlight c++ %}
class Score
{
private:
    int value = 0;
public:
    Score(int value);
    Score sum(int value);
    bool operator==(const Score& s) const
    {
        return this->value == s.value;
    };
};
{%endhighlight %}

Score.cpp

{% highlight c++ %}
Score::Score(int value)
{
    this->value = value;
};

Score Score::sum(int value)
{
    return Score(this->value + value);
};
{%endhighlight %}

Aqui não será necessário relatorar, pois já eliminamos a duplicidade no teste passado. Foi uma alteração mais simples e tudo bem. Podemos seguir para o próximo item do nosso checklist.

Reiniciando o ciclo, temos:

**Passo 1:** Atualizar o checklist

- subtract score
- **equal**
- ~~side-effects~~
- ~~Make “value” private~~
- ~~sum score~~

**Passo 2:** Criar um novo teste para o comparador de igualdade

{% highlight c++ %}
SECTION("test equality") {
    Score score(10);
    Score toCompare(10);
    CHECK(score.qual(toCompare));
}
{% endhighlight %}

**Passo 3:** Execute o teste e veja-o falhar

**Passo 4:** Faça o teste passar.

Score.hpp

{% highlight c++ %}
class Score
{
private:
    int value = 0;
public:
    Score(int value);
    Score sum(int value);
    bool equal(const Score& s) const;
    bool operator==(const Score& s) const
    {
        return equal(s);
    };
};
{%endhighlight %}

Score.cpp

{% highlight c++ %}
Score::Score(int value)
{
    this->value = value;
};

Score Score::sum(int value)
{
    return Score(this->value + value);
};

bool Score::equal(const Score& s) const
{
    return true;
}
{%endhighlight %}

**Passo 5:** Refatore para eliminar duplicidade

Score.cpp

{% highlight c++ %}
Score::Score(int value)
{
    this->value = value;
};

Score Score::sum(int value)
{
    return Score(this->value + value);
};

bool Score::equal(const Score& s) const
{
    return this->value == s.value;
}
{%endhighlight %}

E para finalizar, reiniciamos o ciclo novamente marcando o teste de igualdade como finalizado e destacando nossa última implementação, a subtração de pontos.

Então vamos…

**Passo 1:** Atualizar o checklist

- **subtract score**
- ~~equal~~
- ~~side-effects~~
- ~~Make “value” private~~
- ~~sum score~~

**Passo 2:** Criar um novo teste para o método de subtração de pontos

{% highlight c++ %}
SECTION("test subtract") {
    Score score(100);
    Score newScore = score.subtract(45);
    Score toCompare(55);
    CHECK(newScore == toCompare);
}
{% endhighlight %}

**Passo 3:** Execute o teste e veja-o falhar

**Passo 4:** Faça o teste passar.

Score.hpp

{% highlight c++ %}
class Score
{
private:
    int value = 0;
public:
    Score(int value);
    Score sum(int value);
    Score subtract(int value);
    bool equal(const Score& s) const;
    bool operator==(const Score& s) const
    {
        return equal(s);
    };
};
{%endhighlight %}

Score.cpp

{% highlight c++ %}
Score::Score(int value)
{
    this->value = value;
};

Score Score::sum(int value)
{
    return Score(this->value + value);
};

Score Score::subtract(int value)
{
    return Score(55);
};

bool Score::equal(const Score& s) const
{
    return this->value == s.value;
}
{%endhighlight %}

**Passo 5:** Refatore para eliminar duplicidade

Score.cpp

{% highlight c++ %}
Score::Score(int value)
{
    this->value = value;
};

Score Score::sum(int value)
{
    return Score(this->value + value);
};

Score Score::subtract(int value)
{
    return Score(this->value - value);
};

bool Score::equal(const Score& s) const
{
    return this->value == s.value;
}
{%endhighlight %}

Acho que você já pegou a ideia, certo?

Tenho certeza que você se perguntou “Mas se eu informar um valor negativo ou se a soma/subtração gerasse um valor negativo?”. É claro, se isso for um problema, devemos atualizar nosso checklist adicionando esse ponto para previnir que isso não aconteça e, mais uma vez, executar o ciclo de teste e implementação dessa correção. Mas vou deixar esse exercício para você, caso queria continuar essa implementação.

Basicamente é assim que aplico o TDD na dia a dia. É claro que normalmente encaramos tarefas muitas vezes mais complexas do que um simples sistema de Score, mas o ciclo é exatamente o mesmo, criando um checklist com ainda mais detalhes a serem garantidos no momento do desenvolvimento.

[Access the PR with full code of this example][example-pr]{:target="_blank"}

Espero que tenha gostado e até a próxima!

### Referências:

- Book: Test-Driven Development By Example - Kent Beck
- Book: Refactoring. Improving the Design of Existing Code - Martin Fowler

[example-pr]: https://github.com/guilhermesiani/knowledge-tree-cpp/pull/2/files
