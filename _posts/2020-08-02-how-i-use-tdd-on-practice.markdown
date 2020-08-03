---
layout: post
title:  "How I use TDD in practice"
date:   2020-08-02 14:30:41 -0300
categories: tools
---
What I'll show is an introduction in practice about how to programming using the TDD technique and then you can apply in your programming routine. Although I still recommend to deep with references I'll let on the final of this post.

It is important to highlight that TDD forces us to think about what matters as a final product before coding - and the best of all - with tests, created in a cycle about testing, implementing, and refactoring. This saves us many laps implementing unnecessary things or many refactorings just because we didn't think better at first, preventing bugs, simplifying code design, and getting fast feedback. In short, the focus here is to be productive $$$.

Without further ado. Let's practice!

## Programming with TDD

![TDD Cicle image](/assets/images/tdd-cicle.png "TDD Cicle")
*<center>Image from: https://marsner.com/blog/why-test-driven-development-tdd</center>*

#### Feature Requirements

Our fake feature for this post is a Score System.

#### Tools

- I choose **C++** programming language for this example, but the principles applied will be the same in any other programming language or paradigm.
- We'll use a testing framework called **Catch**, for C++.
- We must write all points that must be covered by tests, so we can't miss a checklist (it can be a simple paper and pencil).

Let's start with:

**Step 1:** Creation of technical requisites checklist, what could be lower or upper-level implementation.

- **sum score**
- subtract score

In this step, we define a checklist and highlighted what we'll work at first, in this case, the sum of the score.

**Step 2:** Create a test for the sum of score implementation.

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

**Step 3:** Run the tests and watch it fail.

In this step, you will note that the tests are our guide for the next implementations, by baby steps, and with much satisfying with tested code.

**Step 4:** Implement the Score class and its sum method to make the test success.

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

**Step 5:** Run the tests again and watch the green bar of success.

The duplicity was purposeful and will be removed on the next step.

**Step 6:** Refactor to remove duplicity.

Score.cpp

{% highlight c++ %}
void Score::sum(int value)
{
    this->value += value;
};
{%endhighlight %}

We could add more tests to ensure the correct behavior of the sum method.

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

Great! We finish the cycle of the implementation for the first technical requisite.

Now let's restart the steps for the next implementation. 

We understand that a Score System needs a way to compare with each other too. Thinking about it we'll increment our checklist with two more things: Equality comparison and to turn *value* private. To turn *value* private is a visibility question that, by security, we decide to prevent that users do changes on this property directly.

So:

**Step 1:** Update the checklist.

- subtract score
- **Make “value” private**
- equal
- ~~sum score~~

**Step 2:** Change the test to perform with private value.

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

**Step 3:** Run the test and watch it fail.

**Step 4:** Make the test pass.

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

**Step 5:** Refactor to remove duplicity.

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

Great! We finish one more cycle.

Seeing the code one more time we note that we have a side-effect risk, just because when we sum score values, we make changes by reference rather create a new instance of Score. Because this we'll update our checklist again and we'll work on it as soon as possible to prevent headaches.

**Step 1:** Update the checklist.

- subtract score
- equal
- **side-effects**
- ~~Make “value” private~~
- ~~sum score~~

**Step 2:** Change the test to ensure that aren't side-effects.

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

We change the second assert to ensure that the initial instance has the same value defined earlier. We are comfortable to create and remove tests and code if it's necessary. It's not reworking. Don't worry!

**Step 3:** Run the test and watch it fail.

**Step 4:** Make the test pass.

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

Here we won't need to refactor. The duplicity was removed on an earlier test. It's a simple change and it's fine for now. We can move on for the next item on the checklist.

Restarting the cycle, we have:

**Step 1:** Update the checklist.

- subtract score
- **equal**
- ~~side-effects~~
- ~~Make “value” private~~
- ~~sum score~~

**Step 2:** Create a new test for equality comparison.

{% highlight c++ %}
SECTION("test equality") {
    Score score(10);
    Score toCompare(10);
    CHECK(score.qual(toCompare));
}
{% endhighlight %}

**Step 3:** Run tests and watch it fail.

**Step 4:** Make the test pass.

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

**Step 5:** Refactor to remove duplicity.

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

And to finish, we restarted the cycle again, turning the equality comparison point done and highlighting our last implementation, the score subtraction.

So let's:

**Step 1:** Update the checklist.

- **subtract score**
- ~~equal~~
- ~~side-effects~~
- ~~Make “value” private~~
- ~~sum score~~

**Step 2:** Create a new test for the score subtraction method.

{% highlight c++ %}
SECTION("test subtract") {
    Score score(100);
    Score newScore = score.subtract(45);
    Score toCompare(55);
    CHECK(newScore == toCompare);
}
{% endhighlight %}

**Step 3:** Run the test and watch it fail.

**Step 4:** Make the test pass.

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

**Step 5:** Refactor to remove duplicity.

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

I think you get the idea, do you?

I'm sure that you are asking yourself "What about I pass negative values or the sum/subtract methods generate a negative value?". Cool! If it's a problem, we'll need to update the checklist adding this topic, prevent that this happens, and more one time, executing a new cycle of test/implementation of this one. I'll let this exercise for you if you want to continue this implementation.

Is that way that I apply TDD on my day-to-day. Surely, we face more complex tasks rather than a simple Score System like that, but the cycle is the same, creating a more detailed checklist to be programmed.

[Access the Pull Request with the full code of this example][example-pr]{:target="_blank"}

I hope you liked it and until the next post!

### References:

- Book: Test-Driven Development By Example - Kent Beck
- Book: Refactoring. Improving the Design of Existing Code - Martin Fowler

[example-pr]: https://github.com/guilhermesiani/knowledge-tree-cpp/pull/2/files
