# GraalVM

- GraalVM is a standalone Java Development Kit to execute Java or JVM-based languages (e.g. Scala, Kotlin), dynamic languages (e.g. JavaScript, R, Ruby, R, Python), LLVM-based languages (e.g. C and C++) in one shared runtime, and supports Linux and macOS platforms on x86 64-bit systems.

![Graal VM Architecture](https://www.graalvm.org/docs/img/graalvm_architecture.png)

- **Graal is a high-performance JIT compiler written in Java.** It accepts the JVM bytecode and produces the machine code.

  ![](https://image.slidesharecdn.com/graalvm-performance-interoperability-181001081748/95/techevent-graalvm-performance-interoperability-9-638.jpg?cb=1538381975)

- Advantages:

  - Safety: No crashes but exceptions instead and no real memory leaks. 
  - A good IDE support: Able to use debuggers or profilers or other convenient tools.
  - It implements the new JVM Compiler Interface – JVMCI to communicate with the VM.
  
  
  
  ## JVMCI (Java Virtual Machine Compiler Interface) & JIT
  
  - **JIT Compiler** : To run the Java program, the JVM interprets the bytecode. Interpreters are slower than native code running on a real processor, so the JVM at runtime can also run another compiler, this time compiling your bytecode to the machine code that your processor can actually run. A JIT-compiler just needs to be able to accept JVM bytecode and produce machine code. It takes as input a `byte[]` and returns a `byte[]`. 
  
    ![JIT Compiler](https://aboullaite.me/content/images/2017/08/jit.png)
  
  - The **JVMCI** is part of the OpenJDK since JDK 9, so we can use any standard OpenJDK or Oracle JDK to run Graal. What JVMCI actually allows us to do is to **exclude the standard tiered compilation and plug in our brand new compiler (i.e. Graal)** without the need of changing anything in the JVM.
  
    ```java
    interface JVMCICompiler {
        byte[] compileMethod(byte[] bytecode);
    }
    ```
  
  ![](https://image.slidesharecdn.com/polyglotonthejvmwithgraalenglish-170521104613/95/polyglot-on-the-jvm-with-graal-english-7-1024.jpg)
  
  ## What does GraalVM offer?
  
  - For _Java_ Programs: GraalVM instead of original JVMCICompiler
  
    - **Faster**: Runs in the context of OpenJDK with new just-in-time (**JIT**) compilation technology.
  
    - **Extensible Application**: GraalVM enables running JavaScript, R, Python, or Ruby in the context of Java applications.
  
    - **Native Image**:  Java bytecode into native images to achieve faster startup and smaller footprint.
  
      ![](https://cdn-images-1.medium.com/max/1600/1*310NJCyl0QF6aPDbBxsbBA.png)
  
  - For *Node.js* Programs: GraalVM instead of V8 Engine
  
    - **Reuse libraries**: Use of existing Java libraries or Java frameworks (like Spark or Flink) directly from Node.js.
  
    - **Large Heaps**:  The V8 JavaScript engine included in the standard Node.js distribution is tuned for browser configurations and designed to work effectively in small heap scenarios. We can run *Node.js* with the heap management of the JVM i.e. the ability for large heap configurations and suitable garbage collection settings.
  
    - **Data Structures in C/C++**: Native data structures can be directly accessed and the compiler can inline across any boundary. This can be beneficial in a scenario where efficient data structures are managed and allocated in C while other parts of the application are written in Node.js. 
  
      ![](https://image.slidesharecdn.com/polyglotonthejvmwithgraalenglish-170521104613/95/polyglot-on-the-jvm-with-graal-english-16-1024.jpg?cb=1495364615)
  
      
  
  - Twitter uses GraalVM in production ([Twitter’s Quest for a Wholly Graal Runtime](https://www.youtube.com/watch?v=G-vlQaPMAxg))
  
    <blockquote class="twitter-tweet" data-lang="en"><p lang="en"  dir="ltr">&quot;Twitter is one of the companies deploying GraalVM in production already today for executing their Scala-based microservices&quot; <a href="https://t.co/EKkZLqVCnn">https://t.co/EKkZLqVCnn</a></p>&mdash; Scala (@scala_lang) <a href="https://twitter.com/scala_lang/status/986297173708066816?ref_src=twsrc%5Etfw">April 17, 2018</a></blockquote>
    <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
  
  ## GraalVM supports Polyglot Programming
  
  ![](https://image.slidesharecdn.com/polyglotonthejvmwithgraalenglish-170521104613/95/polyglot-on-the-jvm-with-graal-english-14-1024.jpg)
  
  - Write polyglot applications with a seamless way to pass values from one language to another
  
  - No copying or [marshalling]("Marshalling is the process of transforming the memory representation of an object to a data format suitable for storage or transmission, and it is typically used when data must be moved between different parts of a computer program or from one program to another") necessary
  
  - High performance when language boundaries are crossed. Most of the time, no additional cost for crossing a language boundary at all.
  
    **Examples**
  
    1. Base Language: `Java`  | Foreign Language: `JavaScript`
  
       ```java
       import org.graalvm.polyglot.*;
       
       class Polyglot {
           public static void main(String[] args) {
               Context polyglot = Context.create();
               Value array = polyglot.eval("js", "[1,2,42,4]");
               int result = array.getArrayElement(2).asInt();
               System.out.println(result);
           }
       }
       ```
  
       
  
    2. Base Language: `JavaScript`  | Foreign Language: `Java`
  
       ```javascript
       var BigInteger = Java.type('java.math.BigInteger');
       console.log(BigInteger.valueOf(2).pow(100).toString(106));
       ```
       
       And execute it in the following way:
       
       ```bash
       $ js --jvm polyglot.js
         10000000000000000000000000
       ```
       
       
  
  ## How does Graal implement interoperability of languages? Introducing Truffle
  
  **Truffle** is a framework for implementing languages using nothing more than a simple abstract syntax tree interpreter.
  
  For creating a language, we need a Grammar. The Grammar is a description of the syntax rules of your language (Syntactical Analysis Phase). It is used to build a parser (or a parse tree). A derivative construct of the parse tree called an *abstract* *syntax tree* (AST)*,* is a natural way to express a program.  
  
  ![](https://ruslanspivak.com/lsbasi-part7/lsbasi_part7_ast_01.png)
  
  ### If Truffle is a framework for writing AST interpreters, then Graal is the thing that makes them fast.
  
  When Truffle is paired with its sister project Graal, it allows those interpreters to be converted into JIT compiling VMs … automatically, resulting to peak performance runtimes competitive with the best hand-tuned language-specific compilers on the market. E.g. the TruffleJS engine which implements JavaScript is competitive with V8 in benchmarks. 
  
  ![](https://image.slidesharecdn.com/polyglotonthejvmwithgraalenglish-170521104613/95/polyglot-on-the-jvm-with-graal-english-20-1024.jpg?cb=1495364615)
  
  **Graal** leverages Truffle as follows:
  
  2. It can run either just-in-time or ahead-of-time (native).
  3. Recognises interpreters written using Truffle and can convert Truffle ASTs into optimised native code.
  
  
  
  ## Truffle and Polyglot
  
  1. Truffle provides a language interop framework called __Polyglot__ which allows Truffle languages to call each other. Truffle implements something called **Object Storage Model** which standardises and optimises the behaviour of dynamically typed objects and JVM bytecode-based languages (Java, Scala and Kotlin).
  2. As Truffle provides a standard framework for expressing nodes in an **AST**, calling into another language doesn’t involve any complex hand-written binding layers. Instead, invoking a function simply joins the ASTs of the two languages together. Those two ASTs are then compiled and optimised by Graal as a single unit, meaning any complexity introduced by crossing the language barrier can be analysed and eliminated.
  
  
  
  ## GraalVM Native Image
  
  - GraalVM can compile Java bytecode into native images to achieve faster startup and smaller footprint for your applications. **GraalVM Native Image** allows you to ahead-of-time compile Java code to a standalone executable, called a **native image**.
  
    ​													`gu install native-image`
  
    
  
    
  
    ![](https://image.slidesharecdn.com/graalvm-performance-interoperability-181001081748/95/techevent-graalvm-performance-interoperability-10-1024.jpg?cb=1538381975)
  
  - This executable includes the application, the libraries, the JDK and does not run on the Java VM, but includes necessary components like memory management and thread scheduling from a different virtual machine, called **Substrate VM**.
  
    ![](https://image.slidesharecdn.com/graalvm-performance-interoperability-181001081748/95/techevent-graalvm-performance-interoperability-28-638.jpg?cb=1538381975)
  
  - The resulting program has faster startup time and lower runtime memory overhead compared to a Java VM.
  
    ```java
    // HelloWorld.java
    public class HelloWorld {
      public static void main(String[] args) {
        System.out.println("Hello, World!");
      }
    }
    ```
  
    Compile the class to bytecode and build the native image. Invoke the executable file.
  
    ```bash
    $ javac HelloWorld.java
    $ native-image HelloWorld
    
    $ ./helloworld
    Hello, World!
    ```
  
  - GraalVM Native Image Generator also enables us to use polyglot capabilities. The native image is much faster than running the same code on the JVM directly. ([Source](https://www.graalvm.org/docs/getting-started/#native-images "Native Images in GraalVM"))
  
  
  
  ## References
  
  1. [Graal & Truffle by Mike Hearn](https://blog.plan99.net/graal-truffle-134d8f28fb69 "Graal & Truffle")
  2. [GraalVM Docs](https://www.graalvm.org/docs/graalvm-as-a-platform "Graal VM Documentation")
  3. [Understanding How Graal Works - a Java JIT Compiler Written in Java by Chris Seaton](https://chrisseaton.com/truffleruby/jokerconf17 "Understanding How Graal Works - a Java JIT Compiler Written in Java by Chris Seaton")
  4. [Deep Dive Into the New Java JIT Compiler – Graal](https://www.baeldung.com/graal-java-jit-compiler "Deep Dive Into the New Java JIT Compiler – Graal")
  5. [Polyglot with Graal](https://www.slideshare.net/akihironishikawa/polyglot-on-the-jvm-with-graal-english)
  6. [TechEvent Graal(VM) Performance Interoperability](https://www.slideshare.net/trivadis/techevent-graalvm-performance-interoperability)
  
