---
author: michael
title: Go Controversial 2 - Would generics improve the Go language?
layout: blog
number: 3
tags:
- golang
- generics
published: false
---

<p>
Generics seem to have gotten a necessity for some people so they won't use a language until they are implemented. Others are put off because of limitations or mistakes in the implementation of generics and won't use a language that implements generics. 
</p>

<p>
While I don't belong to any of those groups there is one claim I don't understand: <b>you don't need generics</b>. Ignoring the abuse of the word need in this context, looking into the go language reference tells me a completely different story. The builtin type map is a generic type and also some builtin functions like copy or append are generic. So the language designers actually felt the need for generics at least for a subset of the language.
</p>

<p>
I'm not fond of restricting users of a language while implicitly admitting that there are use cases where generics make it much safer to use a construct. Now the question is why the designers couldn't decide on generics or why they think that any code that would be based on generics should be revised.
</p>

<h2>What problems do generics solve?</h2>

<p>
For me there are two different kind of generics. Firstly, there is the algorithmic kind and secondly, there is the storage kind.
</p>

<h3>Algorithmic Generics</h3>

<p>
	
</p>
