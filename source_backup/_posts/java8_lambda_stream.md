title: Lambda and Stream in Java 8
date: 2014-08-28 20:05:57
tags: java
---
### 1. Introduction

 This article introduces the new functional-programming features supported in Java 8. It first talks about the basic ways to implement lambdas and streams in the latest version of JDK, and then, pros and cons of this new feature are analyzed. 

 We assume you are already familiar with functional programming styles and have dealt with lambdas and streams in other languages. So in this article we are not going to introduce these things in conceptual level, and some of them are compared with Scheme.

### 2. What is a lambda and a stream in Java 8?

#### 2.1 Lambda expression

Java 8 provides a new package `java.util.function` containing functional interfaces, and a lambda expression is an instance of a subtype of `Object` implementing one of the functional interfaces. Consider this as 'Syntactic Sugar' mapping lambda expressions syntaxes to an anonymous class. 

For example:

* Old Java style using anonymous inner class

```
Thread t1 = new Thread(new Runnable() {     
            public void run() {
                System.out.println("Hey there anonymous!");
            }
        });
```

* Java 8 lambda style

```
Thread t2 = new Thread(
                    ()->{System.out.println("Hey lambda!");} 
                );
```

The syntax of a lambda function is like "()->{...}", where the argument are put in brackets, followed by an arrow alike sign "->", with the body of the lambda as the last part. 

There are following categories [1] of interfaces declared in `java.util.function` package:

* **Function**: Takes a single parameter, returns result based on parameter value
* **Predicate**: Takes a single parameter, returns a boolean result based on parameter value
* **BiFunction**: Takes two parameters, returns result based on parameter value
* **Supplier**: Takes no parameters, returns a result
* **Consumer**: Takes a single parameter with no return (void)

Beside these, one can also declare their own functional interfaces. Generally, the new java compiler accepts all interfaces declared with exactly one method as a functional interface, but one can also explicitly declare one by adding `@FunctionalInterface` annotation. For example:

```
public class Test1 {
    @FunctionalInterface  // This is optional.
    public interface Discount { double apply(double originalPrice); }
    
    Discount discount;
    public void setDiscount(Discount d){ discount = d; }
    
    public double getPriceAfterDiscount(double price){ return discount.apply(price); }

    public static void main(String[] args) {
        Test1 test1 = new Test1();
        double applePrice = 1.0;

        // With type inference, compiler will expand this lambda to 
        // an instance of anonymous class implementing Discount interface
        test1.setDiscount(p -> 0.8*p); 

        double applePriceWithDiscount = test1. getPriceAfterDiscount(applePrice);
        System.out.println(applePriceWithDiscount); // 0.8
    }
    
}
```

#### 2.2 Stream

Stream API is provided in `java.util.stream` package. Stream is built on the basis of lambda with lazy evaluation feature. It is an enhancement made to collection interfaces. It is extremely handy for processing data.

Due to backward compatibility reasons, simply adding additional methods to an interface will require rewriting all the classes implementing this interface. So in order to add additional features to old `Collection` Class, Java 8 introduces `default method`. `Default method` is an default implementation of a method, and is defined in the original class of the interface. With the help of `default methods`, subtypes of `Collection` class, classes user defined with old version Java, can work without realizing the added interfaces explicitly, so that adding new features to old classes will be invisible to users.

`Stream` is a sequence of elements supporting sequential and parallel aggregate operations. It's not a data structure so it doesn't store anything. It takes a lambda expression as an argument and returns another `Stream`, therefore, it can be made into cascades or pipelines.

Using `Stream` is very intuitive:

```
IntStream.iterate(0, i->i+1)
            .filter(i->i>5)
            .limit(3)
            .forEach(System.out::println);
// Output:
// 6
// 7
// 8
```



### 3. How to implement 

#### 3.1 Using Lambda expressions in Java 8

* Shortest way

```
() -> 5 // Without input variable, return a number
x -> 2 * x // Take 1 input variable, return result of multipling by 2
( x, y ) -> x - y // Take 2 input variables, return their substraction
```

* With type specification

```
(int x, int y) -> x + y // Take 2 integers and return their sum
(String s) -> System.out.println(s) // Take 1 String input and return nothing 
```

* Using functional interface

```
@FunctionalInterface
public interface Discount {
    public double apply(double originalPrice);
}

public static void main(String[] args) {
    Discount discount = (x) -> 0.7 * x;
    System.out.println(discount.apply(100)); // 70.0
}
```

* Using Function type

```
java.util.function.Function<Double, Double> discountFunction = (x) -> 0.6*x;
```

#### 3.2 Using Stream APIs in Java 8

For operating with `Stream` APIs, there are 3 kinds of methods:

* **Creating a stream**: `Stream.iterate()`, `Stream.generate()`, `Stream.of()`, `IntStream.of()`, `Array.asList().stream()`, `list.stream()`, `Array.stream()`, `String.chars()`

* **Intermediate Operations**: `peek()`, `map()`, `filter()`, `sorted()`, `limit()`


* **Terminal Operations**: `reduce()`, `count()`, `forEach()`, `match()`, `findFirst()`

The first kind initially generate a stream from an existing `Collection` instance or from a pair of `seed` and `function`.

Intermediate operations accept lambda expression as arguments and return a new `Stream` with "processed" elements. It must be followed by a terminal operation, other wise, due to laziness feature, it will not be executed.

Terminal operations either doesn't return anything or return a new instance of `Collection`.

Example:

```
IntStream.iterate(0, i->i+1)
            .filter(i->i>5)
            .limit(3)
            .forEach(System.out::println);
// Output:
// 6
// 7
// 8
```
Here, it creates an int stream by assigning an initial value with a lambda expression, which is executed to update the value due to lazy evaluation. Then the stream is passed to an intermediate operation `filter`, which filters out the values that doesn't return `True` in the predicate lambda expression. After that, it is passed to another intermediate operation `limit`, which stops the stream from generating new values once it receives 3 values. At the end, it uses a terminal operation `forEach` to do the follow up operation printing out each element it receives.

Note that primitive types should be carefully used in Java `Stream`.

For example[2]:

```
final Integer[] integers = {1, 2, 3};
final int[]     ints     = {1, 2, 3};

Stream.of(integers).forEach(System.out::println); //That works just fine
Stream.of(ints).forEach(System.out::println);     //That doesn't
IntStream.of(ints).forEach(System.out::println);  //Have to use IntStream instead
```

This is because in `Stream` package, Java APIs are defined using generic types `Stream<T>` which only accepts a Class type. Luckily, there're special APIs to specifically deal with primitive types: `IntStream`, `LongStream` and `DoubleStream`. But also be carefull using these primitive type streams. For example:

```
IntStream.range(0, 10).collect(Collectors.toList());
// Cannot compile...

IntStream.range(0, 10).boxed().collect(Collectors.toList());
// Works!
```

Primitive types must be boxed to Objects in order to follow up by a `collect` operation.

#### 3.3 Comparing with Scheme

* `(car list)`, get the head of a list:

```
List<Integer> list = Arrays.asList(0, 1, 2, 3, 4);

Integer car = list.stream().findFirst().get();
System.out.println("Car: " + car); // Car: 0
```

* `(cdr list)`, get the rest of a list:

```
List<Integer> cdr = list.stream().skip(1).collect(Collectors.toList());
System.out.println("Cdr: " + cdr); // Cdr: [1, 2, 3, 4]
```
* `(stream-car stream)`, get the first element of a stream:

```
Integer stream_car = list.stream().findFirst().get();
System.out.println(stream_car); // 0
```

* `(stream-cdr stream)`, get the rest of of the stream:

```
Stream s = list.stream();
s.forEach(System.out::print); // 01234
Stream stream_cdr1 = s.skip(1); // Error: stream has already been operated upon or closed

Stream s2 = list.stream();
Stream stream_cdr2 = s2.skip(1); // Didn't end with a terminal operation, skip() is not computed
stream_cdr2.forEach(System.out::print); // 1234
```

* `(cons 'a '(b c d))`, construct a list with a new head attached:

```
List<String> list = Arrays.asList("b", "c", "d");
List<String> cons_list = Stream.concat(Stream.of("a"), list.stream()).collect(Collectors.toList());
System.out.println(cons_list); // [a, b, c, d]
```

* `(cons-stream n stream)`, construct a stream with a new head attached:

```
Stream cons_stream = Stream.concat(Stream.of(n), list.stream());
cons_stream.forEach(System.out::print); // abcd
```



### 4. Hightlights

* Simplified syntax

Using `Lambda` and `Stream` APIs can reduce the complexity of code. This is pretty obvious.

* Easy to parallelize

By using `Stream` APIs, one can easily parallelize the process simply by changing `stream()` to `parallelStream()`. For example:

```
// Generate a list with 10000000 elements
List<Integer> list = IntStream.range(0, 10000000)
                            .boxed().collect(Collectors.toList());

// Sequencial
long startTime = System.currentTimeMillis();
List<Integer> evenList = list.stream()
                            .filter(i->i%2==0).collect(Collectors.toList());
long estimatedTime = System.currentTimeMillis() - startTime;

// Parallelized
long startTime2 = System.currentTimeMillis();
List<Integer> oddList  = list.parallelStream()
                            .filter(i->i%2!=0).collect(Collectors.toList());
long estimatedTime2 = System.currentTimeMillis() - startTime2;
System.out.println("Time1: " + estimatedTime + " Time2: " + estimatedTime2);

// Output:
// Time1: 1987 Time2: 1029
```

If the process doesn't involve any synchronized variable or operation (here the collect operation is working on one list, therefore need to be locked and unlocked), the improvement would be much significant. The parallel processes is managed under the hood, which means one cannot assign explicitly how many threads to run. But in this way the code has more "cross platform" benifits, so that one don't have change the parallel settings since everything is handled automatically.

* Laziness

Harnessing the benifit of lazy evaluation will help to improve the efficiency of the program. One can define a lot of `Stream` but as long as it's not followed by a terminal operation, it's not executed. Furthermore, due to laziness feature of `Stream`, datas are generated "on the fly" instead of creating a whole list of elements and then iterate through them, not to mention `List` cannot be infinent, which a `Stream` can achieve easily.

### 5. Limitations

* Functional interface limitation

As mentioned before, lambdas in Java 8 are "Syntactic Sugars" to map "arrow style" to anonymous inner class definations. All lambdas must has a functional interface with matching signature. This is due to Java's static type limitation. Although `java.util.function` package has provided a variaty of functional interfaces with generic types and compiler can use type inference to find the matching interface, one has to define his own interfaces if they're not provided, for example, a lambda with 3 input arguments.

* External variable referencing

Java does not allow changing the value or reference of a outside variable inside an anonymous inner class, and all variables that declared outside the anonymous class are inexplictly restrained to `final` inside. So this regulation also applies to lambdas.

For example:

```
int a = 1;
                
Thread t1 = new Thread(new Runnable() {     
    public void run() {
        a = 2; // This cannot pass the compilation, because `a` is declared 
        // inexplictly `final` inside, therefore can't be changed.
    }
});

Thread t2 = new Thread(
        ()->{a = 2;} ); // This cannot pass either.
```

### 6. References

1: http://www.ibm.com/developerworks/java/library/j-java8lambdas/index.html

2: http://stackoverflow.com/questions/23007422/using-streams-with-primitives-data-types-and-corresponding-wrappers
