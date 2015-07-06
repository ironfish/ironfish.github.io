---
layout: post
title: "The Immutable Message in Akka"
category:
- akka
- distributed
- java
permalink: the-immutable-message-in-akka
meta_description: 'The Immutable Message in Akka'
browser_title: 'The Immutable Message in Akka'
---

**Immutable transference** is the only safe way to share state. If a state structure is immutable, then it is impossible for anyone to alter it. In [Akka](http://akka.io), this is how we guarantee the message we send is the same message our recipient receives. By making our messages immutable, we assure their accuracy and eliminate the possibility of contention. Let's take a look at two ways we can model messages in Java.

## Immutable Message

`ImmutableHello` Message in Java

{% highlight java %}
public final class ImmutableHello {
    private final String name;

    private ImmutableHello() {}

    public ImmutableHello(String name) {
      this.name = name;
    }

    public String getName() {
      return name;
    }
}
{% endhighlight %}

* The class is marked `final` so it cannot be extended.
* Prefix the class with `Immutable`, this is the convention, not required but good practice.
* Class variable marked `private final`, enforcing local-only access and immutability.
* `private` no-arg constructor prevents instantiation without a class variable.
* Class constructor requires class variable for creation.
* Getters only.

## Alternative

There is another way to model a Java class to enforce immutability, but some regard it with disdain as it does not follow accepted practices for Java class design. Personally, I like it as it is succinct.

`ImmutableHello` Message in Java (Alternative)

{% highlight java %}
public final class ImmutableHello {
    public final String name;

    private ImmutableHello() {}

    public ImmutableHello(String name) {
      this.name = name;
    }
}
{% endhighlight %}

* The class is marked `final` so it cannot be extended.
* Prefix the class with `Immutable`, this is the convention, not required but good practice.
* Class variable marked `final`, enforcing immutability.
* `private` no-arg constructor prevents instantiation without a class variable.
* Class constructor requires class variable for creation.

## equals, toString and hashCode

The other issue one must consider when modeling immutable classes in Java for messaging is equality. We might desire to compare the message received to another for validation purposes or what have you. As a result, `equals`, `hashcode`, and `toString` implementations are in order.

{% highlight java %}
@Override
public String toString(){
    return "ImmutableHello{name=" + name + "}";
}

@Override
public boolean equals(Object o){
    if (o == this) return true;
    if (o instanceof ImmutableHello) {
        ImmutableHello that = (ImmutableHello) o;
        return (this.name.equals(that.name));
    }
    return false;
}

@Override
public int hashCode(){
    int h = 1;
    h *= 1000003;
    h ^= name.hashCode();
    return h;
}
{% endhighlight %}
