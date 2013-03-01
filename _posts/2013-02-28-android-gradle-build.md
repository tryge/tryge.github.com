---
author: michael
title: Using the new Android Build System and TDD
layout: blog
number: 2
tags:
- android
- gradle
- robolectric
- TDD
---
<p>
Today I ported our android build to use the new gradle based build system. As we heavily employ Test Driven Development I also wanted to be able to test drive our application without resorting to test on devices - that proved to be a challenge.
</p>

<h2>The new android build system</h2>

<p>
It is still in development and I guess far from being released, nevertheless it seems very promising to us as we build almost all of our Java-based software with gradle. The default setup is very simple, there are only 4 things to remember:
</p>
<ul>
	<li>the android plugin must be on the classpath for the build process</li>
	<li>the project must apply the android plugin</li>
	<li>you have to put the sources in the correct directories</li>
	<li>you have to tell the android plugin where the android sdk is</li>
</ul>
<p>
The following snippet shows an example <code>build.gradle</code> like the one published in the <a href="http://tools.android.com/tech-docs/new-build-system/user-guide">official documentation</a>:
</p>

{% highlight groovy %}

buildscript {
	repositories {
		mavenCentral()
	}

	dependencies {
		classpath 'com.android.tools.build:gradle:0.3'
	}
}

apply plugin: 'android'

android {
	compileSdkVersion 15
}

{% endhighlight %}

<p>
You should also notice that the directory structure is different from a standard android project. Basically you have to put all the sources and resources into <code>src/main</code>. Device tests have to go into <code>src/instrumentTest</code>.
</p>

<p>
Finally create a new file <code>local.properties</code> and put something like: <code>sdk.dir=/usr/local/android-sdk/r21.1</code> into it. Please replace the path with your own.
</p>

<h2>Adding a task for unit tests</h2>

<p>
The android plugin doesn't have a task for standard unit tests which is quite unfortunate, therefore we have to add that. First we have to add a <code>sourceSet</code> for our test. Reading the documentation we discover that the sourceSets are configured in the <code>android</code> section. Naturally you would think you could just add an additional <code>sourceSet</code>, but the following doesn't work.
</p>

{% highlight groovy %}

android {
	sourceSets {
		test {
			java.srcDir file('src/test/java')
			resources.srcDir file('src/test/resources')
		}
	}
}

{% endhighlight %}

<p>
First it seems to work, but when you try to use it:
</p>

{% highlight groovy %}

task unitTest(type:Test, dependsOn: assemble) {
        description = "run unit tests"
        testClassesDir = project.android.sourceSetsContainer.test.output.classesDir
        classpath = project.android.sourceSetsContainer.test.runtimeClasspath
}

{% endhighlight %}

<p>
It just tells you:
</p>

<code style="color: red;">Could not find property 'output' on source set test.</code>

<p>
In other words, we can find the source set, but we cannot access the directory where the classes are - and therefore cannot run the tests.
</p>

<h3>Hacky workaround</h3>

<p>
I wasn't in the mood to give up, probably there is another way to access the correct directories but here is what I came up with:
</p>

{% highlight groovy %}
apply plugin: 'android'

android {
	compileSdkVersion 15
}

sourceSets {
	unitTest {
		java.srcDir file('src/test/java')
		resources.srcDir file('src/test/resources')
	}
}

configurations {
	unitTestCompile.extendsFrom runtime
	unitTestRuntime.extendsFrom unitTestCompile
}

dependencies {
	unitTestCompile files("$project.buildDir/classes/release")
}

task unitTest(type:Test, dependsOn: assemble) {
	description = "run unit tests"
	testClassesDir = project.sourceSets.unitTest.output.classesDir
	classpath = project.sourceSets.unitTest.runtimeClasspath
}

check.dependsOn unitTest

{% endhighlight %}

<p>
We define a standard java sourceSet, this works as the android plugin uses the <code>JavaBasePlugin</code>. In the dependencies section we declare our dependency to the application code - as you might see, this is the hacky part and I will search for a better solution in the next few days. If there is one I will post an update. Last we define our <code>Test</code> task and declare it to be executed in the check phase.
</p>

<h2>Robolectric Tests</h2>

<p>
This gives us unit tests, but it doesn't give us the capabilities to test source code that depends on the android API. Of course that doesn't make a lot of sense therefore we will integrate <a href="http://pivotal.github.com/robolectric/">Robolectric</a> to drive our tests.
</p>

<p>
First we add additional dependencies to our dependencies section so it looks like this:
</p>

{% highlight groovy %}
repositories {
	mavenCentral()
}

dependencies {
	unitTestCompile files("$project.buildDir/classes/release")
	unitTestCompile 'junit:junit:4.10', 'org.mockito:mockito-core:1.9.0'
	unitTestCompile 'com.google.android:android:4.0.1.2'
	unitTestCompile 'com.pivotallabs:robolectric:1.2'
	configurations.unitTestCompile.exclude group: 'com.google.android.maps'
}
{% endhighlight %}

<p>
We just add a dependency on the android API and robolectric and exclude the maps API which isn't necessary if you don't use it. Otherwise you just have to download it yourself as google doesn't permit its upload to maven central. <i>Don't forget to declare a repository where gradle can find the dependencies.</i>
</p>

<h3>Writing the Test</h3>

<p>
As our project structure is now different from the standard android project structure, we have to tell Robolectric where our <code>AndroidManifest.xml</code> and the other resources are. To do that we define our own runner:
</p>

{% highlight java %}

import com.xtremelabs.robolectric.RobolectricTestRunner;
import org.junit.runners.model.InitializationError;
import java.io.File;

public class HelloWorldRunner extends RobolectricTestRunner {
	public HelloWorldRunner(Class testClass) throws InitializationError {
		super(testClass, new File("src/main"));
	}
}

{% endhighlight %}

<p>
With this runner we can write a simple test case:
</p>

{% highlight java %}

import org.junit.Test;
import org.junit.runner.RunWith;

@RunWith(HelloWorldRunner.class)
public class HelloWorldTest {
	@Test
	public void testInstantiation() {
		new HelloWorldActivity();
	}
}

{% endhighlight %}

<p>
I know it doesn't look like much, but try to run this testcase without Robolectric and you will see that it is really necessary.
</p>

<h2>Conclusion</h2>

<p>
Gradle is a great build system and we could easily extend the android plugin with ordinary unit tests. I think it is the right choice for the new android build tool and I am looking forward to using it.
</p>

<p>
<a href="/downloads/hello-world-nab.zip">Download</a> the whole example project.
</p>
