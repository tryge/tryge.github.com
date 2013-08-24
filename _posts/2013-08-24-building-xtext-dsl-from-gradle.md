---
author: michael
title: Building your Xtext DSL with Gradle
layout: blog
number: 5
tags:
- eclipse
- xtext
- mwe
- headless
- gradle
- build
keywords:
- eclipse
- xtext
- mwe
- headless
- gradle
- build
---
<p> 
	Xtext is almost what I expect from a tool that is designed to help with designing custom languages. I say almost, because it is heavily incorporated into eclipse IDE/Structures/etc. and therefore brings its own kind of problems. In this post I describe a sample gradle build that lets me compile my xtext languages outside of eclipse's grasp.
</p>

<h2> The example project </h2>

<p>
	I used the 'MyDsl' stub that eclipse uses to initialize an xtext project i.e. our grammar looks like this:
</p>

{% highlight %}
grammar org.xtext.example.mydsl.MyDsl with org.eclipse.xtext.common.Terminals

generate myDsl "http://www.xtext.org/example/mydsl/MyDsl"

Model:
	greetings+=Greeting*;
	
Greeting:
	'Hello' name=ID '!';
{% endhighlight %}

<p>
	Then I wrote my MWE generator script that uses a different structure than eclipse does.
</p>

{% highlight %}
module org.xtext.example.mydsl.GenerateMyDsl

import org.eclipse.emf.mwe.utils.*
import org.eclipse.xtext.generator.*

var grammarURI = "classpath:/org/xtext/example/mydsl/MyDsl.xtext"
var projectName = "mydsl"
var runtimeProject = "../${projectName}"

Workflow {
    bean = StandaloneSetup {
    	scanClassPath = true
    	platformUri = "${runtimeProject}/.."
    }
    
    component = DirectoryCleaner {
    	directory = "${runtimeProject}/src/main/gen"
    }
    
    component = Generator {
    	pathRtProject = runtimeProject
    	projectNameRt = projectName
    	srcPath = "/src/main/java"
    	srcGenPath = "/build/xtext-gen"
    	
    	language = auto-inject {
    		uri = grammarURI
    
    		fragment = grammarAccess.GrammarAccessFragment auto-inject {
				xmlVersion = "1.0"
			}
    		fragment = ecore.EcoreGeneratorFragment auto-inject {
    		}
    		fragment = serializer.SerializerFragment auto-inject {
    			generateStub = false
    		}
    		fragment = resourceFactory.ResourceFactoryFragment auto-inject {
    		}
    		fragment = parser.antlr.XtextAntlrGeneratorFragment auto-inject {
    		}
    		fragment = validation.JavaValidatorFragment auto-inject {
    		}
    	}
    }
}
{% endhighlight %}

<p>
	Anyone who has gone through the experience of customizing the generated mwe script should find his way with ease. However, for those of you who haven't had the pleaseure I'll briefly explain the differences that I introduced.
</p>

<h3>No generation of UI and test sources</h3>
<p>
	As I don't use eclipse I don't value the ui plugin and the test sources are not really helpful, they are more like a hint how you would do it on your own. Moreover it would complicate the setup extremely (at least for the tests) as xtext wants to generate the tests into its own project. You could configure the test project as your dsl project, however, then the test sources would be generated into the same directories as your application source.
</p>

<h3>Change structure where the source files are read and generated</h3>
<p>
	The generated source lives in the build folder, as gradle suggests it should. Your main source folder lives in <code>src/main/java</code> of your project, but that you can change according to your preferences.
</p>

<h3>Generate an XMI file instead of the new default xtextbin</h3>
<p>
	I mainly included this change for you to see how it is done, so if you stumble upon a <code>"missing resource /org/xtext/example/mydsl/MyDsl.xmi"</code> then eclipse has probably upgraded the xtext version in your IDE and generates a <code>MyDsl.xtextbin</code> instead.
</p>

<h3>Use a java validator fragment instead of the default xtend generated one</h3>
<p>
	Using a java validator instead of an xtend one is just a move of convenience. I didn't want to compile xtend sources with gradle but still show that we can directly generate java helper classes like the validator (its practically the same for scoping).
</p>

<h3>The gradle build</h3>

<p>
	I'll leave out the dependencies part (with one exception), I wouldn't expect any reaction other than <code>"those are a lot of dependencies"</code>. The exception is that we have to put our source folder (or the folder where you decide to put your *.xtext files) on the classpath. Xtext searches for the <code>xtext</code> file referenced from the <code>mwe</code> file on the classpath, therefore we have to provide it in this way.
</p>

{% highlight groovy %}
apply plugin: 'java'

repositories {
	flatDir name: 'local', dirs: 'libs'
}

configurations {
	xtext {
		extendsFrom compile
	}
}

sourceSets.main {
	output.dir(new File(buildDir, "xtext-res"), builtBy: 'processLang')
}

dependencies {
	// compile dependencies ...

	xtext files("src/main/java")
	// other xtext dependencies
}

task(generateLang, type: JavaExec) {
	inputs.file new File("src/main/java/org/xtext/example/mydsl/GenerateMyDsl.mwe2")
	inputs.file new File("src/main/java/org/xtext/example/mydsl/MyDsl.xtext")
	outputs.dir new File(buildDir, "xtext-gen")

	standardOutput = new OutputStream() { public void write(int i) {} }
	classpath configurations.xtext
	main = "org.eclipse.emf.mwe2.launch.runtime.Mwe2Launcher"
	args "src/main/java/org/xtext/example/mydsl/GenerateMyDsl.mwe2"
	
	doFirst {
		def projectFile = file(".project")
		projectFile << "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n"
		projectFile << "<projectDescription>"
		projectFile << "<name>" + project.name + "</name>"
		projectFile << "</projectDescription>"
	}
	doLast {
		def projectFile = file(".project")
		projectFile.delete()
		
		def pluginFile = file("plugin.xml")
		pluginFile.delete()
	}
}

task(processLang, dependsOn: generateLang, type: Copy) {
	from "${buildDir}/xtext-gen"
	into "${buildDir}/xtext-res"

	exclude "**/*.java"
	exclude "**/*.g"
}

compileJava.source generateLang.outputs.files, sourceSets.main.java
compileJava.dependsOn processLang
{% endhighlight %}

<p>
	Basically we just define a new <code>JavaExec</code> task, that is a task that executes a java program. We define our inputs to be our <code>xtext</code> and <code>mwe</code> files and our outputs to be in the <code>build/xtext-gen</code> folder. We set the classpath to the dependencies that we defined in the xtext configuration and provide our <code>mwe</code> file as an argument to the program.
</p>

<p>
	That would have been the whole story if it weren't for the fact that eclipse is *cough* well eclipse... so we do some extra work.
</p>

<h3>Generate eclipse project file</h3>
<p>
	When I first saw the message : <code>java.io.IOException: The path '/mydsl/build/xtext-gen/org/xtext/example/mydsl/mydsl/Greeting.java' is unmapped</code> I was puzzled. Of course it wasn't there, your task is to generate it! After a few hours of frustrated searching and looking through the source code I stumbled upon the solution by accident. I was retracing my steps as it had already worked and it wasn't until I deleted the <code>.project</code> file that I had to really breath very slowly and keep calm on purpose. After digging deeper it seems that xtext doesn't know the folder our project lies in because it uses emf resources that is a logical representation of the platform resources hiding their actual physical location. Actually I'm not surprised at all, it just wouldn't have been enough that you have to configure the project name and location several times in the <code>mwe</code> file, it just needed to forget the location of the project. The worst part is that it knows that it knows no location of *any* project but keeps quiet about it.
</p>

<p>
	After you know why the <code>.project</code> file is important, you know why it's beeing generated and deleted afterwards again. We also delete the <code>plugin.xml</code> file that is automatically generated (and honestly I didn't invest the time search for a way to deactivate the generation).
</p>

<p>
	The copy task makes sure that the java files don't and the tokens file does end up in the distributed jar file.
</p>

<p>
	Finally we add the generated java files be compiled by the <code>compileJava</code> task and let the <code>compileJava</code> task depend on our processLang task.
</p>

<h2> Conclusion </h2>

<p>
	If Xtext wasn't so involved with how eclipse projects are setup (and it isn't really difficult to make something like that configurable) and the dependencies would be more manageable (just look at the dependencies in the project) it would be the ideal tool for language development. Unfortunately this is not the case, but gradle gives us the possibility to circumvent most of the obstacles - but not all of them, e.g. I've hidden an exception that the <code>META-INF/MANIFEST.MF</code> file is not found in swallowing the complete standardOutput.
</p>

<p>
	The sources of the complete project can be <a href="/downloads/mydsl.zip">downloaded here</a>.
</p>
