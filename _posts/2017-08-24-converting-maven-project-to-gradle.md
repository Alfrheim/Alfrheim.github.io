---
layout: post
title:  "Convert a Maven project to Gradle"
banner_image: programming.png
date:   2017-08-24 02:16:01 +0200
category: programming 
tags: [maven, gradle, java]
---

Recently I continued a small project I had long forgotten. It was build using maven. Was perfectly fine since I have seen a teammate working with gradle. He challenged me to do it in gradle and I thought... "Why not?".

After a fast checking in google I found quite surprising how fast you can convert a small project to use gradle.

{% highlight bash %}
  # gradle init
{% endhighlight %}

If you want to create too the java project structure automatically just run:

{% highlight bash %}
  # gradle init --type java-library
{% endhighlight %}

This will generate **build.gradle**, **settings.gradle**, **gradlew** and **gradle.bat**.
If your project generates a jar with dependencies file this will not be enought.
To make your project generate a fat jar, you will need to add in **build.gradle**:

> build.gradle
{% highlight gradle %}
...
task fatJar(type: Jar) {
	manifest {
        attributes 'Implementation-Title': 'Recovered project',
        	'Implementation-Version': version,
        	'Main-Class': 'io.alfrheim.Main'
    }
    baseName = project.name + '-standalone'
    from { configurations.compile.collect { it.isDirectory() ? it : zipTree(it) } }
    with jar
}
...
{% endhighlight %}

Now to generate the jar we only need to run:

{% highlight bash %}
  # gradle fatJar
{% endhighlight %}

And the jar will be generated under the **build/lib** folder.
