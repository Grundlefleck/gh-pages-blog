---
layout : post
title : A Quest For Easier Value Objects In Java, Part 1 - Configuring Google's AutoValue
tags:
 - boilerplate
 - Java
 - IDE
 - Eclipse
 - IntelliJ
 - Maven
 - Gradle
---

Despite my numerous gripes about [working day-to-day in Scala](http://grundlefleck.github.io/2013/06/23/using-scala-will-make-you-less-productive.html), one of the things I do love about it are case classes. The ability to define an immutable class which will have `equals()` and `hashCode()` methods, a decent `toString()`, accessor methods (just say no to setters, kids) in a single line of code, is such a breath of fresh air compared to Java. Bleh, all that boilerplate just for a value object with a few fields.

Well, a team at Google think they have a decent answer for this very problem in Java: [AutoValue]h(ttps://github.com/google/auto/tree/master/value). 

AutoValue is designed to take the boilerplate out of writing [value objects](http://martinfowler.com/bliki/ValueObject.html). Using Java's [standardised annotation processing mechanism](https://jcp.org/en/jsr/detail?id=269), you define a class with the properties you want, and a code generator produces all that boilerplate for you. You don't have to write that crap, but even better, you don't have to read it either.

Could it be anywhere near competing with Scala's case classes? To find out, I created a simple project to see what realistic usage of the library might look like. This rest of this post describes how to set a realistic project up to use AutoValue, and first impressions on how the IDEs interact with it. I'm not yet at the point where I can make an informed comparison to case classes.

Before getting into a ton of detail of the features of AutoValue, it was important to me that it is easy:

  * for the project to run with standard build tools, without an IDE
  * for developers new to the project, to get it checked out and usable in their IDE of choice
  * for modification of the code to be immediately recognised in their IDE

In terms of build tools, this meant checking it can work with Maven and Gradle, for IDEs it meant Eclipse and IntelliJ. 

Unlike other tools in this space, such as [Project Lombok](http://projectlombok.org/), AutoValue uses the Java standard annotation processing framework defined in JSR 269. This limits how powerful the AutoValue library can be, but it also means there's not much in the build process that's really specific to it. In theory this experience should hold for all annotation processors.

### The code

I defined one Java class, a super-simplistic represention of a car:

    @AutoValue
    public abstract class Car {
      public abstract String model();
      public abstract int numWheels();
    }

`@AutoValue` tells the annotation processor to take this class, and generate the value type, with all the boilerplate. The generator spits out a Java source file, in `AutoValue_Car.java`. I haven't included the source here, for brevity, and because I think we've looked at enough of that kind of boilerplate for a lifetime. If you want to confirm the generated source is sensible, you can see what was generated [here](https://gist.github.com/Grundlefleck/192b7acb49bbceb5d2cb). The generated class is a package-private subclass, with `equals()`, `hashCode()`, `toString()` implemented in a reasonable way, saving you the hassle. Once that's generated, you then have to define a static factory method in the abstract class `Car`, returning new instances of the generated subclass. A little bit of hoop-jumpiness, but nothing too insane. 

I also added a single JUnit test, with a single assertion: `assertImmutable(AutoValue_Car.class);`. Using my own [Mutability Detector](www.mutabilitydetector.org) library, this unit test performs static analysis on the generated class, to check it's immutable. Not only is it a good sanity check to highlight any surprises in how AutoValue generates immutable classes, but it also makes sure that the generate and compile portion of the build is working correctly. Note I can only reference `AutoValue_Car.class` because I put my test in the same package.

Now that I had a class annotated with `@AutoValue`, and an expectation of what would be generated, and a way to check compilation worked, I set about seeing how such a setup would be configured across the build tools and IDEs.

### Maven

I was pleasantly surprised by Maven (bet you don't hear that often). I just added this dependency:

    <dependency>
        <groupId>com.google.auto.value</groupId>
        <artifactId>auto-value</artifactId>
        <version>1.0-rc1</version>
        <scope>provided</scope>
    </dependency>

The project compiled, and the tests were successful. There was literally nothing else that had to be done to make sure `@AutoValue` was picked up by the annotation processor. Win. This may be a factor of how annotation processing is integrated to `javac`, but still, it's nice when something Just Works&trade;.

### Gradle

Again, it Just Worked&trade;: it compiled and executed tests successfully. Nice part about this case was the dependency declaration was shorter without really losing information: `compile 'com.google.auto.value:auto-value:1.0-rc1'`, which is a nice aspect of Gradle.


Now we have the kind of build configuration that would allow us to run tests on CI, it was time to move on to the IDEs. My expectation is that a developer should be able to checkout the project, import it into their IDE, and have it compile successfully, ready to be worked on. 

### Eclipse

For Eclipse, to be able to import a project, I expect to run a command from the build tool to generate the config files. The commands I used were `mvn eclipse:eclipse` and `gradle eclipse`. Unfortunately here is where the "just work-iness" ended. 

Eclipse requires some configuration to tell it that it should run annotation processors in a project. This can be set in the project properties through Eclipse, and I was able to work it out through the GUI without too much furrowing of brow. However, I wanted to get to a point where a new developer could be up-and-running straight away. I never got to that point with either Maven or Gradle. The closest I got was to add two config files to source control (`.factorypath` and `.settings/org.eclipse.jdt.apt.core.prefs`). Fortunately in both cases the config files are machine-independent, so it's pretty safe to share them with your fellow developers.

Maven also required the following snippet to be added to the `maven-compiler-plugin`'s `<configuration>` section.

    <annotationProcessors>
        <annotationProcessor>com.google.auto.value.processor.AutoValueProcessor</annotationProcessor>
    </annotationProcessors>

There was also a weird issue I'm blaming on Eclipse. Even with the correct config files, when I imported the project, I needed to turn annotation processing [off and on again](https://www.youtube.com/watch?v=p85xwZ_OLX0) to get it to work. Nasty.

Overall, not a glowing testimonial for Eclipse or Maven and Gradle's support for it.

### IntelliJ

For IntelliJ, as well as the build commands `mvn idea:idea` and `gradle idea`, I also attempted to import the project directly from IntelliJ. In this case, Maven beat Gradle. The project compiled and ran the tests successfully both from using `mvn idea:idea`, and by importing the Maven "external model" from within IntelliJ. However, for the equivalents in Gradle, I'm still unable to find the right configuration to get it to build successfully. However, this could just be a general Gradle+IntelliJ problem: the resultant project configuration just doesn't look right, regardless of annotation processors. It could be that I'm using fairly recent version of Gradle, and the IntelliJ plugin hasn't caught up, or that I'm just less familiar with the Gradle+IntelliJ combination than Maven+Eclipse, and I'm missing something obvious. Answers on a postcard please.


## Conclusion
Apart from the Gradle+IntelliJ combination, annotation processing seems to be well supported across build tools and IDEs. You may have an opinion on code generation, but when your build tool successfully builds without much configuration, and your IDE reflects changes and generates code seamlessly, it's not that different from a vanilla compiler. Initial impressions are good.

Hopefully in a later post I'll be able to comment on AutoValue's feature set in detail, but for now I just wanted to see what the basic usage felt like. Time will tell whether I would consider AutoValue a reasonable compromise in the quest for something like Scala's case classes for Java.


Entire source code listing can be found in this GitHub project: [https://github.com/Grundlefleck/autovalue-example](https://github.com/Grundlefleck/autovalue-example)
