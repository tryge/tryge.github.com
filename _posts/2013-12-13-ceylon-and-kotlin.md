---
author: michael
title: The next programming language? Ceylon and Kotlin
layout: blog
number: 6
tags:
- ceylon
- kotlin
- jvm
- modern
- language
- design
keywords:
- ceylon
- kotlin
- jvm
- modern
- language
- design
---
<p>
	There are already a lot of resources that make <b>some</b> comparison between ceylon, kotlin and Xs, where Xs stands for
	some JVM-based languages. I didn't include these in this post as I haven't spent enough time to analyze them in detail or
	they expose some major issues (regarding design, philosophy, readability, etc.) from my point of view.
</p>

<p>
	I probably have to stress that I don't say <i>Xs are flawed or bad</i>, <b>they're just not right for me</b> or I haven't spent
	enough time to evaluate them.
</p>

<p>
	Also I should note that this post is about my feeling to code in these languages and whether I could use them for products where a Java base already exists.
</p>

<h2>Why am I looking for a new language?</h2>

<p>
	If you ask someone why they are looking for a new language, they'll probably mention something like <i>boilerplate</i>, <i>expressiveness</i> or
	<i>productivity</i>. I'm not one of those guys:
	<ul>
		<li> boilerplate: yes,  in Java you write a lot of boilerplate code, but it is semi-automated by your IDE.</li>
		<li> expressiveness: what does expressiveness exactly mean to you? I have always been able to express my design intentions in Java, so it can't be the expressive power - most likely you mean concise code and readability issues, however, I do think that Java code can be written in a readable way. And that you can write unreadable code in any non-trivial language.</li>
		<li> productivity in a language: I sometimes read people claiming that they were almost instantly (a few days) more productive in a new language. How is this possible? To read up on the enormous API of todays languages takes weeks or months. In Java I don't think about how to do a specific task, I just do it. In a new language I have to learn how to do it, then I have to review whether there are better ways to express the idea or if there are hidden glitches. To cover the whole range of APIs and development tools that I have to use for my development tasks at work (from embedded development, to android app development, to desktop IDE-like tools, to web service and web application development) it takes at least 6 months until I <b>can</b> get as productive as I am in Java right now - with (about) the same quality of my code. However I believe it is much longer than that.</li>
	</ul>
</p>

<p>
	So why am I looking for a new language?
</p>

<p>
	Firstly, I'm continuously looking for new languages out of interest. I like to see new concepts and ideas - and often enough these ideas inspire me to improve something in the products I work on in Java. That is, because it is a concept and even though it might not have support for syntactic sugar it is expressible in Java and might be worth implementing.
</p>

<p>
	Secondly, there are issues with Java that I simply can't ignore: <br><br>
	Often enough we run into problems because of incompatibilities of libraries as one transitive dependency is library X version 1 and another is library X version 2, of course they are incompatible. For some time we used an OSGi container to circumvent the problem, however there is overhead in managing the dependencies (e.g. not all Java libraries are osgified), so the solution was not ideal (for us). Instead, we restrict the libraries that we use, which works well, even though sometimes developers complain that 'it' would be much easier with library 'X'.<br><br>
	Another issue is repository management, build systems and continuous integration. There are <b>no</b> <i>standard tools</i>. So if I need to replace an administrator I know I will have to plan at least 3 months of training. Worse, if something happens during this time it could cost us a lot of development time as our developers had to support the new team member. <br><br>
	Deployment and the Update of our products are another hot topic, but I don't expect it to be better with a new language (it would really be a surprise), so I'm not going into it. I wanted to note it down though.<br><br>
	This starts trivial: there are some features of Java that have knowingly been implemented in a suboptimal way because of backwards compatibility issues - additional to the usual design flaws that creep into a large system. I won't list them here, <a href="http://marksweep.blogspot.co.at/2011/10/java-8-backwards-compatibility.html">they</a> <a href="http://c2.com/cgi/wiki?JavaDesignFlaws">were</a> <a href="http://chrononsystems.com/blog/java-7-design-flaw-leads-to-huge-backward-step-for-the-jvm">covered</a> <a href="http://www.javapuzzlers.com/">enough</a>. However the design flaws themselves are not the issue, <b><i>the issue is the current strategy</i></b> of how to deal with them - which is to do nothing. I'll probably work for another 50 years (yes, I am a workaholic...), if I keep true to my current view of things I'll always be coding (at least one third of my week), so even if Java is the new Cobol it won't be usable in 20 years.
</p>

<p>
	Lastly, even though I previously <b>was</b> hard on the boilerplate/expressiveness/productivity argument, it <b><i>is</i></b> nice to have a clean, regular and immediately understandable language. These reasons are just not driving my cause.
</p>

<h2> First Impressions of Kotlin and Ceylon </h2>

<p>
	After spending a lot of time reading about the languages I implemented a simple project in both Ceylon and Kotlin. The most surprising thing for me was that even though the languages share a lot of ideas they have a completely different feel.
</p>

<p>
	Considering that Ceylon is about 18 months 'older' than Kotlin (they were announced only 6 months apart, however, RedHat had been working on it for 'almost two years' and Jetbrains for only 'almost a year') it is understandable that they are in a different stage of development. So don't take my comments regarding Kotlin too seriously. Also Jetbrains has made it clear, that they are in the process of designing the language and if something doesn't work they change it.
</p>

<h3>Kotlin</h3>

<p>
	One thing I noticed almost immediately is that Kotlin (at M6.2) is by far not finished. Some parts of the API are not consistent, e.g. for Java lists there are Extension Properties <code>first</code> and <code>head</code> as well as Extension methods <code>first()</code> and <code>firstOrNull()</code>. My opinion is that there shouldn't be a multitude of methods how to perform the exact same <i>trivial</i> task. But it also shows that there is no coherent design in place. I have the feeling that it is missing (or not published in any way) and that the current code does not express this design (if it exists).
</p>

<p>
	One more thing is that there is a real lack of introductory material on Kotlin. There is documentation, but it is in a very rough state and it doesn't make the vision of Kotlin clear. I don't know what I can expect from the language. For example, if there hadn't been a <a href="http://medianetwork.oracle.com/video/player/2623518144001">talk</a> about reflection I'd be completely in the dark whether reflection for Kotlin was supported or I'd have to 'decode' the JVM structures. Also I'd like to know what to expect in M7 and M8 (so at least two milestones in advance).
</p>

<p>
	However, it does make a few topics very clear:
	<ul>
		<li>No compromises in Java compatibility. This is one of the things that drastically separates Kotlin from Ceylon. While it is possible in Ceylon to call Java-Code, the other way round it seems to be more difficult (when creating a ceylon project, the IDE has an <i><b>option (sic!)</b></i> to enable Java code calling Ceylon code and a warning that this might affect performance)</li>
		<li>Compilation Speed matters, that is, Kotlin code should compile at least as fast as Java code (it doesn't at the moment).</li>
		<li>Safe &amp; Concise completes the mantra.</li>
	</ul>
	Especially the first point piqued my interest in Kotlin.
</p>

<p> One thing I have to remark is that Kotlin has a very familiar feeling, aside from syntax and API differences it still feels somewhat similar to Java coding - which is a good thing. I feel that all my experience with Java is helping me to write Kotlin code.</p>

<h3>Ceylon</h3>

<p>
	Ceylon's documentation is overwhelmingly complete and readable, I can only recommend reading the <a href="http://ceylon-lang.org/documentation/1.0/tour/">Tour of Ceylon</a>. Also the language itself feels complete and consistent, however, it really feels like a new and completely different language. I'd say that the barrier of using an existing Java library in Ceylon is higher than in Kotlin.
</p>

<p>
	What I find most impressive and intriguing about the language is the concept of union and intersection types. In a recent <a href="http://ceylon-lang.org/blog/2013/12/13/three-legged-elephants/">blog post</a> Gavin King said that Ceylon had the most satisfying solution for combining <i>subtype polymorphism</i> with <i>parametric polymorphism</i>. I completely agree with this statement. I like the language not only for their solution of <i>generics</i>, but also because it seems like a lot of consideration has been invested in every single feature of the language. This just makes the language very harmonic.
</p>

<p>
	Also infrastructure has been taken into account:
	<ul>
		<li>Modularity is built into the language</li>
		<li>Dependencies are built into the language</li>
		<li>There is a standard repository management system</li>
		<li>It has a great meta-model of the language (event though I haven't found out how legacy Java modules are supported in the meta-model)</li>
		<li>Simple, regular and powerful visibilities using a single annotation <code>shared</code></li>
	</ul>
</p>

<p>
	The concepts left a really good feeling about the language, however, there is one thing that is bugging me. My application took ages to compile! In the IDE it wasn't so apparent as I haven't used Eclipse for a long time and still remember Eclipse being really slow - but it might just be that it was the compiler that slowed things down. I really thought that I did something wrong when I built the whole application on the command line for the first time. I actually aborted with CTRL+C before I realized that it actually might take <b>this</b> long to compile.
</p>

<h2>Direct Comparison</h2>

<p>
	I generally have to say that I like both languages a lot. Ceylon's design is much cleaner but Kotlin has a very familiar feel when coding (and regarding its design, it's not finished yet). Where Ceylon has brought OOP type systems to a whole new level, Kotlin concentrates on the most important pieces that improve upon Java like null-safety. Both languages allow me to 'overload' a predefined set of operators and provide a way to define hierarchical structures easily. In the latter case I must say that I like Kotlin's solution of using anonymous extension methods more that the construction based approach taken by Ceylon. According to the <a href="http://ceylon-lang.org/documentation/1.0/faq/language-design/#extension_methods">Ceylon FAQ</a> extension methods might make it into a subsequent release, so my hopes are still up that I get more flexibility for defining my hierarchical structures.
</p>

<p>
	It took me longer to get used to Ceylon's keywords and also its rhythm as they are really different compared to Java, but I was very satisfied with my result. In contrast, I caught Kotlin's syntax faster but the result didn't feel as satisfying. Probably that is because it felt <i>easier</i> so I didn't have the same sense of accomplishment. In my Kotlin application I mostly used classes from the Java Platform as Kotlin's stdlib is not really complete yet whereas Ceylon provides their own language module where I needed to lookup <b>a lot of</b> things - like how to append a list to another one and similar trivialities.
</p>

<p>
	What I really like about Kotlin is that I could start using it in any of my projects right now without a fuzz. There are a <a href="http://confluence.jetbrains.com/display/Kotlin/Kotlin+Build+Tools#KotlinBuildTools-Gradle">gradle plugin</a>, a <a href="http://confluence.jetbrains.com/display/Kotlin/Kotlin+Build+Tools#KotlinBuildTools-Maven">maven plugin</a> and <a href="http://confluence.jetbrains.com/display/Kotlin/Kotlin+Build+Tools#KotlinBuildTools-Ant">ant tasks</a> for build integration and Kotlin even works on Android which is a big plus. It's different with Ceylon, I couldn't just add Ceylon code to my existing projects to add a new feature, from what I gather now (correct me if I'm wrong), I'd have to transform the modules to Ceylon modules where I could mix Ceylon with Java - IF I can do this with my build system. There is <a href="https://groups.google.com/d/msg/ceylon-users/2vEeWgoNyZ0/LaNY7X75KlMJ">proof</a> that it is possible to run Ceylon on Android, however it doesn't give me the confidence to use it in my projects right now.
</p>

<p>
	What came as a surprise to me is that the Ceylon-IDE is a much nicer IDE than the Kotlin plugin for IntelliJ. I thought that Jetbrains who specializes in IDE's would invest more time in their plugin, but probably they are waiting until the final design decisions of the language have been made. As soon as Jetbrains starts to dogfood Kotlin I'm sure the plugin will get better, however for the time being I prefer the Ceylon-IDE.
</p>

<h2> Conclusion </h2>

<p>
	I'm a little dissatisfied with my findings, while I like both languages I am not able to use any of them now. On the one hand I have a really nice and elegant language that I can't use in my projects as it is not integrated in the toolchain and IDE; we use IntelliJ and we'll never change our Java IDE to Eclipse so we won't mix our code. On the other hand I have a language that I could use in my projects but which is unfinished and thus the risk of using it is too high (and also there are a lot of rough edges to it that need to be fixed before investing more time).
</p>

<p>
	If I'd start a new project, I'd at least investigate whether I could use Ceylon for it.
</p>
