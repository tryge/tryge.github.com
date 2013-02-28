---
author: michael
title: Go Controversial 1 - What about proper Constructors?
layout: blog
number: 1
tags:
- golang
---

<p>
When I first started programming in go, I found it irritating that there is no custom initialization phase when an "object" is being constructed. As I have heard some complaints about that and I've already made up my mind, I will share my thoughts here. In this article I use the term constructor to refer to the concept as it is used in C++ or Java - not go's concept of constructors (which are factory functions).
</p>

<h2>Initialization</h2>

<p>
Each type has a zero value, e.g. the zero value for <code>int</code> is <code>0</code>. Optimally a type is ready to use with its zero value, but sometimes this isn't enough and we will use composite literals. A composite literal creates a new instance of the specified type and allows one to provide initialization for the fields. Suppose we have the following piece of code:
</p>

{% highlight go %}

type SomeStruct struct {
	A, B string
	C int
	p int
}

{% endhighlight %}

<p>
SomeStruct is a complex type and has 3 exported fields and a private one. To initialize an instance we would write something like this:
</p>

{% highlight go %}

x := SomeStruct{"first", "second", 3}

{% endhighlight %}

<p>
In this way you can initialize all the exported fields of a struct. Now there's a question, how would you fit in a magic function that is automatically called upon instance creation. It obviously wouldn't make much sense to call it before the fields had been initialized, therefore it must be after that.
</p>

<h3>What would such an initialization function do?</h3>
<p>
It would probably allocate data for private fields and do something with the already initialized fields.
</p>

<h3>What if there is an error?</h3>
<p>
The only way to react is to <code>panic</code> as you don't have a return value to pass an error. But that is against the philosophy of go, look at the statement again:
</p>

{% highlight go %}

x := SomeStruct{"first", "second", 3}

{% endhighlight %}

<p>
It does look harmless, doesn't it? We just declared a new variable that is initialized with this composite literal.
</p>

<h3>How can we reason about it?</h3>
<p>Very simple, nothing bad can happen (<i>except</i> if you run out of memory). However, if a magic function was called instead, you could not reason about this statement without looking into the code that implements the type. It could do anything, like opening a connection to the SETI project and wait for them to announce that they have found aliens.
</p>

<p> But there's a more pressing problem, what about the following piece of code:</p>

{% highlight go %}

var x SomeStruct

{% endhighlight %}

<p>
When do you suggest this magic function should be called?
</p>

<h3>But I really do need a constructor!</h3>

<p>
Actually I've never been fond of constructors, they don't really communicate the intent I would like to see. When writing Java code I prefer static factory methods and similarly that's the way to go - just write a factory function. That's how the concept of go constructors are defined. A function that returns an instance of a type is a constructor for that type. It also solves the problem of reasoning, everyone knows that a function call can cause/detect errors in one way or the other.
</p>

<p>I've heard the argument that it is still possible to work with not initialized structures. Firstly, I think that is always possible and secondly, you shouldn't worry about that. You can (should) provide documentation so that everybody knows how to use your API and if you really want to prohibit the use of an uninitialized type, return an interface instead and don't export your actual type.
</p>

<h2>Conclusion</h2>

<p>
The concept of Java/C++ constructors doesn't fit into the philosophy of go, there's a conflict with reasoning about variable declarations and composite literals. If you need initialization, provide a factory method. If you want to prevent the possibility of using your instances uninitialized return an interface and don't export your actual type.
</p>
