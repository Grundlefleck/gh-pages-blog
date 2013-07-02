---
layout : post
title : Using Scala Will Make You Less Productive
tags:
 - Scala
 - Java
 - Productivity
---
... in an uncontrolled study, in which I reflect on my personal experience, I feel like I'm less productive. Therefore, you will be less productive as well, QED. Well, I think I can safely tick the 'science' check box.

In this lengthy post, I'll share my thoughts and opinions on Scala, and moving to it for day-to-day coding. Using a cavalcade of flawed data; logical fallacies; biased opinion; startling lack of citations; and basic nonsense, I will convince you that using Scala will make you less productive. I will finish with an underwhelming and unexciting conclusion. You have been warned.

###Scala Experience To Date###

I first began using Scala professionally in December 2012. Prior to that my exposure had been light: reading [Programming In Scala](http://www.artima.com/shop/programming_in_scala_2ed) and the odd blog entry, and occasionally firing up the REPL to try out examples. Since December I've worked on two projects, which puts me at about the six-month mark. The projects were not exactly revolutionary: single page web apps using [Typesafe](http://typesafe.com/)'s [Play! framework](http://www.playframework.com/); some data entry, some calculations; a MySQL database. Bigger than that last To Do list app you built, smaller than your average Sprawling Monolith Enterprise System&trade;. The point being that I've not used Scala for a pretend, toy app, but also, the projects have not been around long enough to reflect on those development choices that you have to slowly watch unfold over months and years.

### General Experience To Date ###

I've been developing software professionally for a whopping 3 years. The majority of that was in Java and JavaScript, although I did have a few months of Clojure development in a scheduled-for-but-put-on-hold-for-business-reasons production application. Plus also throwing in occasional forays into Bash and Python, or whatever, when I need something quick and dirty that I'll likely throw away. I'm nowhere near the best developer in the world, but nowhere near the worst either.

Within Java I was and am a rigorous [TDD](http://en.wikipedia.org/wiki/Test-driven_development)'er, a fan of [SOLID](http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principles, a proponent of minimizing mutability, a supporter of functional techniques. I like static typing, and I like to find my silly coding mistakes early. Despite my attempt to cultivate a [neckbeard](https://twitter.com/NeckbeardHacker) I have no experience of [Haskell](http://www.haskell.org/haskellwiki/Haskell). What have I done that you should listen to a word I have to say? Nothing really, I haven't found mindshare with a new programming language, or saved trillions of Altairian dollars conceiving the one true web framework, or even worked on something you'll have heard of. But you've somehow ended up reading my ramblings here, so I'd suggest I'm probably not distracting you from curing world hunger, so what have you got to lose?

###The Good Things&trade; With Scala###

It would take the most ardent Java fanboy to claim there is nothing good about Scala. I'm not married to Java, so I don't have to view it through rose-tinted glasses, or defend its honour. I don't really have a dog in this fight. Languages will come and go, and [COBOL developers can still make a living](https://www.google.co.uk/search?q=cobol+jobs), so I'm not personally invested in disparaging Scala for funsies. Of course if you're a zealot, on either side, you're going to go ahead and claim I'm biased, or I have an agenda.

So what good can I see in Scala?

####Lambdas####

... or closures or whatever you want to label "syntax and library support for passing functions around". Although technically that definition is met by Java's anonymous inner class construct, you would have to be a babbling fool not to concede that Scala's syntax and current library support is just plain _better_. The introduction of JDK 8's lambda syntax may make it a more interesting comparison, but for now, Scala's miles ahead. Consider this predicate `( x => x.isInteresting )`: the Java equivalent will contain the declaration of the type of `x` 3 times, and is also likely to be at least 3 lines of Java code with even the most aggressive vertical-space-saving formatting strategy. That's a bunch of boilerplate that has to live somewhere, and you have to train your brain to ignore it.

####Functional Idioms in the Standard Libraries####

Walking hand in hand with lambdas is the support available in the Scala SDK. Want to take a collection of elements, filter by some property, and convert the results into another type? `myStuff.filter(x => x.isInteresting).map(x => toNewThing(x))`[\*](#footnote_1) is right there, available as soon as you're in Scala land. In Java, this can be achieved once you add a library like Guava to your classpath, but as previously mentioned, there will be lot of noise that doesn't add anything. 

I like functional operations. I like filter, map, zip, fold. I like that operation I can only seem to remember as 'groupBy' but probably has a more traditional name. I like immutability and programming without side-effects. I use those operations and program that way today in Java, supplemented by Guava. In Scala it's easier syntax and it's more easily available.


####Type Inference####

There are a lot of places in Scala where you can omit type information. `val x = "Hello"` is a simple example. The compiler is a clever cookie, and realises that you probably mean `val x: String = "Hello"`, so you don't need to bother telling it that. A tiny smattering of type inference exists in Java, but it's not in the same league. 

There's a double-edged sword here, in that now you have the ability to omit type information, you will omit type information, even when it would be useful to include it. Care needs to be taken to make sure that type inference is actually helping to make code clearer. Everyone who commits a chunk of code relying on type inference should be forced to play a game of "Pin The Tail On The Type". First blindfold the author, spin them round a couple of times, then expose them to the code they've just written. If they can't immediately identify all the types necessary to understand what's going on, you get to hit them with sticks until candy falls out. That last bit might be a different game I've mixed in.


####Case Classes####

I bloody love case classes. Again, you'd have to be a couple of patents short of a troll-portfolio to argue that case classes don't provide something that's sorely lacking in Java. If you prefer generating: a constructor; getter methods (forget the setters, immutability ftw); toString and hashCode, then you're a crazy person. 

If like me, you want to use static types in place of state when you can, case classes make it much less painful. Let me explain. Let's say a user, in the UI, enters information to create a new todo item in the database. They enter name, description and due date. So, you make a type that has name, description and due date. You can then validate that input using a type representing what the user enters, e.g. checking the due date is not in the past. But you're saving this in the database, so where's the id field? Well, many people will recommend adding an `Option[String]` field that's None until it's saved. For me, that's not taking advantage of the power of case classes to have appropriate static types. First you have `case class NewBlogEntry(name: String, description: String, dueDate: DateTime)`, and then, once it's validated and saved in the database, and you fetch it back, it becomes an instance of `case class SavedBlogEntry(id: String, name: String, description: String, dueDate: DateTime)`. Here you'd also add other fields that aren't entered directly, e.g. the datetime when the entry was created. "Wait", I hear you ask, "won't that result in an explosion of new classes?" Well, it will definitely result in _more_, but case classes are so cheap syntactically (that snippet of code above is possibly all you need), and cheap enough at runtime, that it's worth it not to have fields that don't apply to the object at particular points in time.

Case classes also supply a `copy()` method that pretty much renders the [Builder Pattern](http://en.wikipedia.org/wiki/Builder_pattern) defunct. I was initially worried that it's only a shallow copy, which may result in subtle bugs if somewhere in the object graph you have a mutable type. In practice, I've yet to observe this, and the bias towards immutability means I don't spend too many sleepless nights fretting about it.

In a hypothetical world where I am offered a choice between introducing either lambdas or case classes to Java, I may find it hard to make the decision, that's how fond I am of case classes. 

###Other Miscellaneous Good Things&trade;###

I won't go into much detail here, but some other good things are:
 * reduced boilerplate in general
 * an eminently reasonable [Benevolent Dictator For Life](https://en.wikipedia.org/wiki/Benevolent_Dictator_for_Life) in [Martin Odersky](https://twitter.com/odersky). I have considered that every time I find something really frustrating about Scala, I'm just going to read a blog post, forum comment, or StackOverflow answer from Scala's BDFL. I've generally found his stance, or explanations, to be so god damn agreeable that I just can't keep up the [caremad](http://www.urbandictionary.com/define.php?term=caremad). 
 * the actor model with Akka, providing a simplified concurrency model. Technically Akka's Actors are accessible from Java as well, but I suspect they'll be less well supported in the Java community (disclaimer: I've not looked).
 * inter op with existing Java libraries. Joda Time is a great example. There's no de facto Scala date and time library because, well, Joda Time is there, accessible, and good (not just good _enough_, actually good).
 
Hopefully I've convinced you that I'm not a foamy-mouthed Scala hater, having spent over 1000 words describing my favourite bits. That should give me a bit of credibility for some of the bitching and moaning that's on it's way.

###The Bad Things&trade; With Scala###

####Compile Time####

Let's just get it out of the way. The amount of time that Scala needs to compile is bad. [XKCD-swivel-chair-swordfighting bad](http://xkcd.com/303/). I find that comic funny because, as I have found with Scala, it's [true](https://twitter.com/Grundlefleck/status/328839382922571776). I never understood those dynamic language guys who preferred tests over static types. Now I get their point. I feel like I could write all the tests I need to cover weaknesses in Java's type system, and have them all run quicker than the Scala compiler does. That feedback loop is important to me. I don't like it when I make a small change, re-run a unit test, and it takes 15 seconds to even start running the test. I'm not working on a big codebase. I have a powerful machine. I have what is considered the best IDE for Scala (IntelliJ). Running in the Play! (well, sbt) console improves this slightly, but still compile time is distractingly long.

Of course, as mentioned, despite slow compile time being supremely frustrating, Martin Odersky is brimming with reasonableness in [describing why](http://stackoverflow.com/a/3612212/4120). Perhaps each time I'm waiting on a compile I can use the time to re-read that post, to keep me calm. 

####The IDEs####

I alluded to this before, but I never realised how much I was giving up in the IDE department when switching from Java to Scala. I thought, well, Scala is a statically typed language, there's no reason the support won't be as good. And I keep hearing Scala advocates saying IDEs are improving leaps and bounds. Well, they might be, but improving on "terrible" doesn't necessarily mean the result is anywhere near "good".

I can already hear the Scalaist's response: "An IDE is proof your language sucks", "Awww, poor you, now you have to write all those getters and setters yourself", "Your IDE is just a tool to generate crap faster". I call bullshit. I rarely generate code. I rarely write getters and setters. There is nothing I can see in Java that needs an IDE half as much as Scala's implicit conversions do. Coding Java with a good IDE means: 
 * renaming variables, methods, types, anything, is a keyboard shortcut and typing the new name. For anything other than local variables in Scala, I don't even bother with the IDE as it will likely mess it up. Instead I just do it manually. The IntelliJ support for renaming functions is more accurate in JavaScript than it is in Scala. Yes, JavaScript. In Java, I don't even think about it, it's accurate and correct in 99.9% of cases.
 * same goes for extracting variables, methods, interfaces. I can forget about the mechanics of refactoring, and focus on improving readability.
 * when I ask "who references this type?", or "who implements this interface?" I get a pretty accurate answer in Java  (reflection and module boundary cases notwithstanding). That's not my experience in Scala with either IntelliJ or the Eclipse plugin. More often than not, I revert to text searches, like I'm in a dynamic language.

And that's about it. That's the most important things an IDE offers me. You could argue "Well, you should just name your thing right the first time, and design your types carefully upfront so they never need to change". You could argue that, but that would be dumb. The point you would be missing is that a lot of the times, the names _were_ right the first time, but now they're not, things have changed. Types _were_ carefully chosen, but now they need to adapt to suit some new, unforeseen requirement, because things have changed. For me, an IDE is not about mindlessly clicking your way through another graphical wizard to spit out XML config documents for Yet Another Framework. Being able to rename and extract is an important factor in keeping the "soft" in "software". IDEs for Scala are not as good as those for Java in the bits that matter.

####Overuse of Operator Overloading####

Yes, yes, it's not operator overloading, it's allowing symbols for method names, totally different. You're right; it's much worse. You can still overload those comfortable, familiar operators, like `+` or `*` to do unexpected things. Not only that, you can use whole raft of unexpected symbols to do unexpected things. Symbols are great when they confer meaning. Over several years in school I learned what many symbols mean: `+`, `-`, `/`, `*`, `~` all have meaning. Seeing `BigDecimal a = x.plus(y)` is a drag compared to `a = x + y`. Brilliant use of the `+` symbol as a method name. But now let's take sbt as an example. Using the `%` symbol to delimit artefact coordinate information is unnecessary and _gratuitous_. Using `%%` to mean, very specifically, "append the version of Scala to the version of this dependency" is a slap in the face to the principle of least astonishment. This, dare I say it, "spiteful" use of symbols as method names appears in several of the libraries I've been working with: 
ScalaQuery; specs2; Akka. Ironically, the least friendly of them all, scalaz, actually gets a pass in my book. It's so thoroughly laden with unfamiliar symbols, like `|@|`, `<$>` and `<<*>>` (all pronounced "pleasejustletmeusehaskell") that they at least serve as a warning. That usage of symbols is fine if your goal is to provide a library with more functional concepts than the Scala SDK. Not cool if you are the de facto standard build tool, that virtually all projects will need. 
 
####Implicit####

The implicit keyword is one of my least favourite Scala language constructs. Take implicit parameters for example. It used to be you would have functions, and they would take arguments. To be able to drop some of the arguments for whatever reason, you could partially apply the function with the arguments you want to leave off. Java's typically verbose version is to have a class reference it's own instance fields in a method (watch the functional weenies writhe in agony over that comparison). Alternatively you could create a function which closed over a variable in scope. Now Scala allows you to have an implicit parameter, which means somewhere in the code there can be a declaration that says "if you're looking for something of type Foo to pass to some method, use me". Then when you invoke the method, if you have that declaration in scope, you don't have to actually reference it to pass it to the method, Scala's compiler will look it up for you. 

So what's my problem? For me, that lookup is opaque, it hides the parameter, kinda like bunging it in a `ThreadLocal` and referencing it further down the call stack. It's more difficult to keep the data flow in your head than it needs to be, and it's pointlessly different from passing an argument to a function. Are there use cases for it? Probably, likely some pretty good ones somewhere. I'm told that the `implicit CanBuildFrom` parameter in the Scala Collections library is solving problems that can't be solved any other way. So fine, I make my peace with the guts of a library doing using implicit parameters, as long as it hides the complexity from me. For those implicit parameters that bleed out into an API, have any of the uses of I've come across been good? No, not yet. Has every example I've seen been a hindrance to readability? Yes.

Conversions are Scala's other use of `implicit`. If a method takes a type `Foo`, and you try to pass in `Bar`, before the compiler tells you to fix your obvious type error, it first tries to discover if there's a way to convert a `Bar` to a `Foo` and use that instead. Similarly if you try to invoke a method on `Foo` that doesn't exist: before raising an error the compiler will look around for any available conversion that can turn `Foo` into something with a matching method. 

So what's my problem? Similar to implicit parameters, it's the opaque, hidden lookup that detracts from readability. The bit where implicit conversions kinda sorta look like a really neat composition syntax (this type `Bar` is really just a `Foo` with an extra method or two) is really enticing. The ["Enrich My Library"](http://www.artima.com/weblogs/viewpost.jsp?thread=179766) pattern is a really useful concept, but it's the easy composition bit that does the legwork, I can't help but think that's where the feature should have ended, the use of `implicit` ruins it for me.

One argument I've seen in favour of Scala's implicit conversions is that "Java also has these conversions, like from int to long, Scala just makes it available". Correct: Scala does make it available. Instead of needing to learn a handful of weird, edge-case ridden, confusing conversions in Java, you have to learn every Tom, Dick and Sally's weird, edge-case ridden, confusion conversion they've added to their library. And boy do people who like to experiment with new languages like to take that availability and run with it. And they run right past it's logical conclusion End Zone. They run through that End Zone like Forrest Gump. Except when they keep running, they cease to be Forrest Gump, they're actually Arnold Schwarzeneger, and are now being violently hunted down in a game show format, because the compiler found "implicit def toRunningMan()" in scope.

If you think that makes no sense, try debugging an accidentally and silently "missed" implicit conversion in ScalaQuery's DSL. 

####The Shallow End/Deep End False Dichotomy####

It has been touted that Scala allows you to wade into the shallow end of the pool, using only the features that you're [comfortable with](http://pages.zeroturnaround.com/rs/zeroturnaround/images/Scala2013-A-Pragmatic-Guide-To-Adoption.pdf). From there, you can progress to the deep end, treading water over more advanced features at your own pace. Sounds good, right? Until you realise that, by accident or not, each library developer pulls you out of the shallow end, throws you straight into the deep end, and stands on your head until the bubbles of program comprehension escaping from your lungs cease to go 'bloop'. As soon as you want to do any production, business web application development, there's a bunch of stuff you're probably going to need to do, for example, route HTTP requests, make database calls, serve HTML and JSON/XML, write tests for it all. Unless you want to sink time into reimplementing this common stuff, you're going to reach for a library that can help. As alluded to previously, the people who like to release libraries for new languages also like to experiment with language features. Unless the goal of their project is to cater to Scala noobs, they'll cram every clever use of Scala into the client facing part of their library. Now is the point where you take a deep breath and hold it; you're in deeper water than you're capable of swimming in.

Of course, you can just use familiar old libraries from Java, but then you're just not "with it", man. You're not, "following the idioms", dude. You're just a crusty old Java fuddy duddy who doesn't "get it", bro. There won't be support, like documentation, for your "dinosaur" ways, fella.

I know, I know, "Whaaaa, poor me, all that stuff I get for free, created during time donated by volunteers, doesn't do what I want", right? Or perhaps "If you don't like it, why don't you contribute to making it better rather than whining?". Well that argument doesn't seem to apply to those who castigate Java and its free and open-source libraries, where pretty much everything has been handed to us without the need to pay or contribute in any way. We can't have it both ways.

###It Doesn't Really Matter###

I've given irrefutable scientific evidence on why Scala will make you less productive. Many will respond with fluff and opinions and many forms of bias, claiming they are already more productive in Scala. That's fine. I'm now going to go ahead and claim it doesn't really matter anyway.

What?! Of course it matters, Scala is so much more concise, there's almost no boilerplate! Think of all that time you won't waste looking at getters and setters and anonymous inner classes! Well, not in my experience. We can probably all agree ["typing is not the bottleneck"](http://sebastianlab.com/post/140303165/typing-is-not-the-bottleneck), well, what about syntax? The syntax of a language hasn't been something I've found to be a big factor in my productivity. Syntax is just a hurdle that you have to jump, and once you've done that, you just kinda move on with your life. Of course, some hurdles are bigger than others, and take more skill to surmount at all, and others are more frequent, and require you to jump more often.  An anoynmous inner class to pass a function around is worse than a lambda. Final fields and constructors that set them are worse than declaring a `val` in the class definition. Getters and setters and equals and hashCode is worse than a case class. But are they bottlenecks? Not really, just minor, albeit constant, annoyances. The paradigm is the key. If functional programming is what you want, you can do it in your language. Understand the techniques, discover why they're useful for you, because *just* switching languages won't buy you _anything_. 

What about all the wonderful things in the standard library? Yep, `filter` and `map` and `fold` operations on collections out of the box is nice. That the SDK uses `Option` instead of `null` is an improvement. These things are nice, but I generally get what I'm looking for by adding some libraries to core Java. Yes, even a library to introduce an `Option` type if I consider it necessary. Would it suit me better if I didn't have to deal with extra libraries? Yes. Does it make up for the other things I don't like about Scala? No.

If I could flick a switch which: levelled me up in Scala; made the compiler fast; made the tools usable; and made the libraries sane, I probably would be more productive. Slightly. But I can't. So I'm investing my finite learning time, being less effective day-to-day, for limited gains. Learning syntax and library APIs are the low-return investments. Paradigms and techniques are where it's at, yo. 


###Ahmdal's Law For Programming Languages###

... or "Grahamdahl's Law" (my name is Graham, see what I did there?)

I think it's time for a variant of Amdahl's Law for programming language. That is, using an improved language has a theoretical limit in the improvement it can give to your overall productivity. 

How much of your time, as a software developer, do you spend doing stuff that is language agnostic? I spend a lot of time doing stuff that doesn't go any faster when I wave the wand of a magic new programming language at it. I spend time figuring out:

 * what a feature is supposed to do, understanding what the user needs, and exploring edge-cases that will have to be handled. 
 * which names and concepts will make it easier for the next developer to understand the business logic.
 * how to iterate new features while maintaining backwards compatibility, both in a long-lived sense such as maintaining APIs, but also in a short-lived sense, such as how to deploy database schema migrations with no downtime.
 * what the most appropriate logging, monitoring and metrics are needed to give us visibility into how our applications are used, and how well they perform.
 * what the performance and scaling characteristics of a design are likely to be, and how to measure if I'm not confident.
 * how to coordinate and integrate and deliver, and generally work, as part of a team.
 
What do these activities have in common? They are issues you spend time on, that remain roughly constant, regardless of programming language used. Let's say delivering a new feature is 50% language agnostic activities, and 50% hands-on-keyboard-coding. That's way more pure coding than I think is representative for me, but nevertheless.  Even if I'm using a language that I can code in twice as well, I've still only saved 25% overall. There is absolutely no way I'm twice as good in Scala as I am in Java, that's unlikely to ever happen. 10% better maybe, one day, at a push, with a following wind. Given how much effort and time I've spent not being as productive as I could be, seems like a poor investment for me, particularly when that piddly 10% is diluted further by Grahamdahl's Law. Underlying assumptions seem to be that a 10x better lanuages equates to 10x overall improvement, and I think that's just plain wrong[\*\*](#footnote_2). 


So... in conclusion. Scala will make you less productive, for some period of time, and if you do get more productive in it, it probably won't matter that much anyway.

Told you the conclusion would be disappointing.



---

<a id="footnote_1">
</a> 
\* Yes, there is terser syntax, I know.

<a id="footnote_2">
</a>
\*\* Please, for the love of $DEITY, can we get some _science_ over here?
