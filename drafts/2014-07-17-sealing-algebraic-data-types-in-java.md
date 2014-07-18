Sealing Algebraic Data Types in Java

Recently I caught up on Dick Wall's Devoxx UK talk, titled "What have the Monads ever done for us?"[0][http://parleys.com/play/53b15afee4b0543940d9e5d8/chapter15/about]. Dick does a great job of introducing terms such as "monoid", "functor" and "monad" in a way that's easy to grasp, even for Java-damaged minds like my own. I thoroughly recommend watching the talk to all developers who have heard these terms and want a simple way to understand it without, trudging through tutorials using silly metaphors like space suits or burritos.

Also, achievement unlocked: met Dick Wall at the speaker's dinner[https://www.flickr.com/photos/125714253@N02/14489008396/in/set-72157645322154176]. A thoroughly, and completely predictably (based on the Java Posse podcasts!) lovely chap.

As well as heartily recommending you check out Dick's talk, I wanted to pick up on something he mentioned, and suggest an approach. Dick introduces an algebraic data type for a simplied linked list implementation, using an abstract class `DeadSimpleList` with exactly two subclasses: `DeadSimpleListNil` and `DeadSimpleListCons`. The base class looks like this:

    public abstract class DeadSimpleList<T> {
       public abstract <U> DeadSimpleList<U> map(Function<T, U> function);
       public abstract boolean isEmpty();
       public abstract String contentsAsString();
    }

(For clarity, most of the useful functions and generics had been ommitted.)

One of the points of these algebraic data types is that subclassing is limited and controlled. As opposed to an interface, they are not intended to be extended by users of the type. For example, Scala's abstract Option type is extended by the concrete Some and None types. Some and None entirely defines Option's behaviour. You want to explicitly prevent someone coming along and adding a new extension of Option, like SomethingWhenIFeelLikeIt, because if it's possible for such a beast to exist, it becomes more difficult to reason about. 

In talking about how extension should be prohibited, Dick says "[Haskell and Scala let you] keep the superclass public, inherit from it, in a controlled way, but then stop anyone else from inheriting from it... and I don't know that there's a good answer for that in Java". 

I think there's a good answer. Or at least, as good as these things can ever be in Java.

As the example continues, the two subclasses are declared as `public class DeadSimpleListNil<T>` and `public class DeadSimpleListCons<T>`, which would have to reside in different files from the superclass. I think the desired trick here, is to limit subclassing by reducing visibility of the abstract class' constructor. Like so:

    public abstract class DeadSimpleList<T> {
      public abstract <U> DeadSimpleList<U> map(Function<T, U> function);
      public abstract boolean isEmpty();
      public abstract String contentsAsString();

     // add this constructor
     private DeadSimpleList() { }

     ...

Adding the private constructor will cause the subclasses to fail to compile, because the constructor of the abstract class will not be visible. The only way to get them to compile is to move the subclasses into a scope where they can invoke the private constructor of the superclass, like so:

    public abstract class DeadSimpleList<T> {
        // abstract methods, private constructor, as before
        
        public static final class DeadSimpleListNil<T> extends DeadSimpleList<T> {
          // method implementations as before
        }
        
        public static final class DeadSimpleListCons<T> extends DeadSimpleList<T> {
          // concrete constructor and method implementations as before
        }
        
        ...
        
Now you have complete control over how the abstract class is subclassed. Users can still reference the inner classes, and construct instances of them. Crucially, they can't create their own subclass called RandomlyEmptyDeadSimpleList, and thus nobody has to waste any brain cycles fretting about the possibility of its existence.

Although the semantics are quite different from Scala's `sealed` keyword, the outcome is roughly equivalent: only when you control the source code of base class can you add subclasses. Surprisingly for Java, this mechanism isn't as annoyingly cumbersome as you might have predicted.


[0] I missed it when I attended Devoxx, since I was giving my talk at the same time on another track. It's common for me to visit conferences where talks I'm interested in clash, this was the first time where one of those talks was mine.
