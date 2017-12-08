title: 【Java 8】Lambda 表达式
date: 2017-12-08 21:07:58
tags: ["Java", "Lambda"]
categories: Java 8
---

## 定义

Lambda(λ) 表达式是一种在 _被调用的位置_ 或者 _作为参数传递给函数的位置_ **定义匿名函数对象** 的简便方法。下面是关于 Lambda 表达式的几个点：
* 匿名（Anonymous） - 不像其他普通方法那样具有名字
* 函数（Function） -  Lambda 表达式不像普通方法那样属于某个特定的类，它是独立于类存在的。但是和方法一样，Lambda 表达式有参数列表、函数主体和返回值，还可能有可以抛出的异常列表。
* 传递（Passed around）- Lambda 表达式可以作为参数传递给方法或者存储在变量中。
* 简洁（Concise）- 无需像匿名类那样写很多的模板代码。

<!-- more -->

下面是一个示例
```java
@FunctionalInterface
interface Calculator {
    int cal(int a, int b);
}

public class HelloWorld {
    public static void main(String[] args) {
        Calculator c = (a, b) -> a + b;
        System.out.println(c.cal(1, 2));
        c = (a, b) -> a * b;
        System.out.println(c.cal(1, 2));
    }
}
```

## Lambda 形式
Lambda 表达式的基本形式如下所示：
```
(argument list) -> code
```
下面是一个例子：

![](/images/lambda/形式.png)

如上所示： Lambda 表达式包含三个部分：
* 参数列表（A list of parameters） - 上图中为 `(Apple a1, Apple a2)`
* 箭头（An arrow） - 把参数列表和 Lambda 主体分隔开
* Lambda 主体（The body of the lambda） - 上图中为 `a1.getWeight().compareTo(a2.getWeight())`，该 Lambda 主体会返回 compareTo 的结果。

Lambda 函数的主体可以是表达式（expression）或者语句（statement），所以 Lambda 函数返回值有下面两种情况：
* 如果 Lambda 主体为表达式，那么 Lambda 函数的返回值就是表达式的计算值
* 如果 Lambda 主体为语句，那么 Lambda 返回值就是语句的返回值

关于语句和表达式的区别，可以参考 [这篇文章](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/expressions.html)，这里简单说一下：假设有一条语句 `int c = a + b;`，那么表达式就是指 `c = a + b`，即不包含 `int` 和 `;`，每个表达式都会有一个计算值（void 也算一种特殊的计算值）。

所以细分一下，Lambda 表达式有两种形式：
```
(parameters) -> expression
```
和（使用大括号）
```
(parameters) -> {statements}
```

下面是 Lambda 表达式的几个例子：

![](/images/lambda/demo.png)

| 使用场景 | 使用示例 |
|:---|:---|
| boolean 表达式 | `(List<String> list) -> list.isEmpty()` |
| 创建对象 | `() -> new Apple(10)` |
| Consuming from an object | `(Apple a) -> { System.out.println(a.getWeight()); }` |
| Select/extract from an object | `(String s) -> s.length()` |
| 合并两个值 | `(int a, int b) -> a * b` |
| 比较两个对象 | `(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())` |



## 函数式接口
我们可以在函数式接口 （Functional interface）中使用 Lambda 表达式。简单来说，函数式接口就是只定义一个抽象方法的接口（接口中可以包含额外的 default 方法）。例如 Comparator 和 Runnable 都是函数式接口：
```java
public  interface Comparator<T> {
    int compare(T o1, T o2);
}

public interface Runnable {
    void run();
}
```
我们一般在接口定义中加上 `@FunctionalInterface` 注解来声明该接口是一个函数式接口。例如下面的形式：
```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```
当一个接口通过 `FunctionalInterface` 被声明为函数式接口时，编译器将会检查接口的合法性，如果接口不合法，会报编译错误。

现在考虑一个问题，Lambda 表达式是如何匹配函数式接口的呢？假设我们有一个如下定义的函数式接口：
```java
@FunctionalInterface
interface Calculator {
    int cal(int a, int b);
}
```
下面是使用 lambda 表达式以及匿名类来创建 Calculator 对象的示例代码。在下面的代码中对象 c 和 c2 的实现是等价的。
```java
public void demo() {
    Calculator c = (int a, int b) -> a + b;

    Calculator c2 = new Calculator() {
        @Override
        public int cal(int a, int b) {
            return a + b;
        }
    };
}
```
从上面的例子中，我们可以看到 **Lambda 表达式** 是和函数式接口中的 **抽象方法** 进行匹配的，其中 Lambda 表达式中参数匹配 cal 方法的参数，Lambda body 的内容作为抽象方法的具体实现，Lambda body 的计算值作为方法的返回值。这也是为什么要求函数式接口只能有一个抽象方法的原因。

函数式接口中抽象方法的签名（signature）描述了 Lambda 表达式的签名，因为 Lambda 表达式并没有名字，所以这里的签名只关注三个方面：**方法参数** 、**返回值** 以及 **异常声明**。我们将抽象方法所描述的 Lambda 形式称为函数描述符（function descriptor）。在 Calculator 类中，cal 方法对应的函数描述符为 `(int, int) -> int`，即接受两个 int 类型作为参数，表达式的计算值为 int 类型。所以下面的 Lambda 表达式都是合法的：
```java
(int a, int b) -> a
(int a, int b) -> a + b
(int a, int b) -> 0
```
如果 Lambda 表达式抛出一个可检查异常，那么对应的抽象方法所声明的 throws 语句也要与之匹配。看下面的一个例子：
```java
@FunctionalInterface
interface ThrowExceptionInterface {
    void run(int a, int b);
}

public class LambdaTest {

    public void throwException() {
        // 这里编译时会报 Unhandled Exception：java.io.Exception
        ThrowExceptionInterface t = (int a, int b) -> {
            throw new IOException();
        };
    }
}
```
其实也很好理解，Lambda body 中的内容会作为抽象方法的具体实现，在方法中抛出了异常但是方法声明中却没有相关的异常声明，编译器肯定要报错的。

另外还有一个特殊的 void 兼容规则。如果抽象方法的返回值为 void，即对应的函数描述符为 `(T) -> void`，那么对于 body 为 **语句表达式（statement expression）** 的 Lambda 表达式，只要求参数列表匹配即可。看下面的例子：
```java
@FunctionalInterface
interface VoidInterface {
    void run(int a);
}

public class LambdaTest {
    public static void voidTest() {
        List<String> list = new ArrayList<>();
        // 这里 a++ 返回一个 int，但是和 void 兼容
        VoidInterface v = (int a) -> a++;
        // 下面的代码会报错，因为 a-1 不是一个语句表达式
        v = (int a) -> a-1;
    }
}
```
这里说一下语句表达式:
> The term “statement expression” or “expression statement” refers to expressions that are also allowed to be used as a statement.

语法表达式有下面四类：
* Assignment expressions
* Any use of ++ or --
* Method invocations
* Object creation expressions

### 类型检查
Lambda 表达式本身并不包含它是实现哪个函数式接口的信息，编译器会根据 Lambda 表达式所处的上下文（context）环境来推断 Lambda 表达式的目标类型（target type），例如对于下面的代码：
```java
Calculator c = (int a, int b) -> a + b;
```
Lambda 表达式会赋值给 Calculator 对象，那么该 Lambda 表达式对应的目标类型就是 Calculator 接口，该接口中的 cal 方法对应的函数描述符为 `(int, int) -> int`，这个和 `(int a, int b) -> a + b` 可以匹配，这样就完成了类型检查。下图是一个完整的例子：

![](/images/lambda/类型检查.png)

### 类型推断
在上面我们提到编译器会根据上下文环境推断出与 Lambda 表达式对应的函数式接口，这意味着编译器同样可以根据接口中抽象方法的函数函数描述符推断出 Lambda 表达式的签名，这样编译器就可以知道 Lambda 表达式的参数类型，这样就可以省略 Lambda 表达式中的参数类型，
```java
Calculator c = (a, b) -> a + b;
// 当只有一个参数时，可以省略掉 ()
VoidInterface v = a -> a++;
```

## Java 8 中的函数式接口
在 Java 8 中定义了一些函数式接口，位于 `java.util.function` 包下，下面是这些接口的总览：
```
+--- BiConsumer.java
+--- BiFunction.java
+--- BinaryOperator.java
+--- BiPredicate.java
+--- BooleanSupplier.java
+--- Consumer.java
+--- DoubleBinaryOperator.java
+--- DoubleConsumer.java
+--- DoubleFunction.java
+--- DoublePredicate.java
+--- DoubleSupplier.java
+--- DoubleToIntFunction.java
+--- DoubleToLongFunction.java
+--- DoubleUnaryOperator.java
+--- Function.java
+--- IntBinaryOperator.java
+--- IntConsumer.java
+--- IntFunction.java
+--- IntPredicate.java
+--- IntSupplier.java
+--- IntToDoubleFunction.java
+--- IntToLongFunction.java
+--- IntUnaryOperator.java
+--- LongBinaryOperator.java
+--- LongConsumer.java
+--- LongFunction.java
+--- LongPredicate.java
+--- LongSupplier.java
+--- LongToDoubleFunction.java
+--- LongToIntFunction.java
+--- LongUnaryOperator.java
+--- ObjDoubleConsumer.java
+--- ObjIntConsumer.java
+--- ObjLongConsumer.java
+--- Predicate.java
+--- Supplier.java
+--- ToDoubleBiFunction.java
+--- ToDoubleFunction.java
+--- ToIntBiFunction.java
+--- ToIntFunction.java
+--- ToLongBiFunction.java
+--- ToLongFunction.java
+--- UnaryOperator.java
```
### Predicate
用来测试对象是否满足某种条件。该接口定义了一个 test 方法，接受一个泛型对象（T），并返回测试结果（boolean），函数描述符为 `T -> boolean`。下面是一个使用示例：
```java
public <T> boolean judge(T t, Predicate<T> p) {
    return p.test(t);
}

public void testPredicate() {
    String text = "111";
    System.out.println(judge(text, s -> s != null));
}
```
下面是 Predicate 接口的实现：
```java
/**
 * Represents a predicate (boolean-valued function) of one argument.
 */
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return true if the input argument matches the predicate,
     * otherwise false
     */
    boolean test(T t);

    /**
     * Returns a composed predicate that represents a short-circuiting logical
     * AND of this predicate and another.  When evaluating the composed
     * predicate, if this predicate is false, then the other
     * predicate is not evaluated.
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    /**
     * Returns a predicate that represents the logical negation of this
     * predicate.
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    /**
     * Returns a composed predicate that represents a short-circuiting logical
     * OR of this predicate and another.  When evaluating the composed
     * predicate, if this predicate is true, then the other
     * predicate is not evaluated.
     */
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    /**
     * Returns a predicate that tests if two arguments are equal according
     * to {@link Objects#equals(Object, Object)}.
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

我们看到在 Predicate 类中，除了 test 方法，还定义了三个 default 方法，`and`, `or` 和 `negate`，它们分别对应逻辑运算中的与（&&）、或（||）、非（!）操作。通过这三个方法，我们可以构造更复杂的 predicate 表达式：
```java
public void testPredicate() {
    String text = "111";
    Predicate<String> a = s - > s != null;
    Predicate<String> b = s - > s.length() > 3;
    System.out.println(judge(text, a.and(b)));
    System.out.println(judge(text, a.negate()));
    System.out.println(judge(text, a.or(b)));
}
```
对应的输出结果为：
```
false
false
true
```
另外 and 和 or 方法是按照在表达式链中的位置，从左向右确定优先级的。因此 `a.or(b).and(c)` 可以看作 `(a || b) && c`。

### BiPredicate
BiPredicate 针对两个参数对象（T, U）进行测试，函数描述符为 `(T, U) -> boolean`。下面是该接口的定义：
```java
/**
 * Represents a predicate (boolean-valued function) of two arguments.  This is
 * the two-arity specialization of {@link Predicate}.
 */
@FunctionalInterface
public interface BiPredicate<T, U> {

    boolean test(T t, U u);

    default BiPredicate<T, U> and(BiPredicate<? super T, ? super U> other) {
        Objects.requireNonNull(other);
        return (T t, U u) -> test(t, u) && other.test(t, u);
    }

    default BiPredicate<T, U> negate() {
        return (T t, U u) -> !test(t, u);
    }

    default BiPredicate<T, U> or(BiPredicate<? super T, ? super U> other) {
        Objects.requireNonNull(other);
        return (T t, U u) -> test(t, u) || other.test(t, u);
    }
}
```
下面是一个使用示例：
```java
public void testBiPredicate() {
    BiPredicate<Integer, Integer> b = (x, y) -> x > 0 && y > 3;
    boolean r = b.test(1, 4);
    System.out.println(r);
}
```

### Consumer
Consumer（消费者），针对对象进行某种操作（消费对象）。该接口定义了一个 accept 方法，会将该方法作用于目标对象，函数描述符为 `T -> void`。下面是使用示例：
```java
public <T> void consume(T t, Consumer<T> c) {
    c.accept(t);
}

@Test
public void testConsume() {
    String text = "1234";
    consume(text, s -> System.out.println(s.substring(2)));
}
```
下面是 Consumer 类的代码
```java
/**
 * Represents an operation that accepts a single input argument and returns no
 * result. Unlike most other functional interfaces, Consumer is expected
 * to operate via side-effects.
 */
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    /**
     * Returns a composed  Consumer that performs, in sequence, this
     * operation followed by the after operation. If performing either
     * operation throws an exception, it is relayed to the caller of the
     * composed operation.  If performing this operation throws an exception,
     * the after operation will not be performed.
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```
Consumer 中定义了一个 andThen 的 default 方法，通过该方法我们可以对目标对象进行链式（chain）处理，下面是一个示例：
```java
public void testConsume() {
    StringBuilder builder = new StringBuilder();
    Consumer <StringBuilder> a = s -> s.append("abcd");
    Consumer <StringBuilder> b = s -> s.reverse();
    Consumer <StringBuilder> c = s -> s.append("1234");
    consume(builder, a.andThen(b).andThen(c));
    System.out.println(builder.toString());
}
```
输出结果为：
```
dcba1234
```

### BiConsumer
BiConsumer 针对两个对象（T, U）进行操作，对应的函数描述符为 `(T, U) -> void`。下面是该接口的定义：
```java
/**
 * Represents an operation that accepts two input arguments and returns no
 * result.  This is the two-arity specialization of  Consumer.
 * Unlike most other functional interfaces, BiConsumer is expected
 * to operate via side-effects.
 */
@FunctionalInterface
public interface BiConsumer<T, U> {

    void accept(T t, U u);

    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
        Objects.requireNonNull(after);

        return (l, r) -> {
            accept(l, r);
            after.accept(l, r);
        };
    }
}
```
下面是一个例子：
```java
public void testBiConsumer() {
    BiConsumer<String, String> b = (x, y) -> System.out.println(x + y);
    b.accept("111", "222");
}
```

### Supplier
Supplier（供应商），返回一个泛型对象（生产对象）。该接口中定义了一个 get 方法，没有方法参数，返回值是一个泛型对象，函数描述符为 `() -> T`。下面是一个使用示例
```java
public <T> T supplier(Supplier<T> s) {
    return s.get();
}

public void testSupplier() {
    String text = supplier(() -> "1111");
    System.out.println(text);
}
```
下面是 Supplier 接口的定义：
```java
/**
 * Represents a supplier of results.
 */
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     */
    T get();
}
```

### Function
Function 接口就相当于 `y=f(x)` 中的函数 f，接收一个 x（argument）返回计算值 y（result）。该接口定义了一个 apply 方法，接收一个 T 类型的对象，返回一个 R 类型的结果，函数描述符为 `T -> R`。下面是一个使用示例：
```java
public <T, R> R func(T t, Function<T, R> f) {
    return f.apply(t);
}

public void testFunction() {
    String text = "1234";
    int i = func(text, t -> Integer.parseInt(t));
    // 输出 1235
    System.out.println(i + 1);
}
```
下面是 Function 接口的定义：
```java
/**
 * Represents a function that accepts one argument and produces a result.
 */
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     */
    R apply(T t);

    /**
     * Returns a composed function that first applies the before
     * function to its input, and then applies this function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    /**
     * Returns a composed function that first applies this function to
     * its input, and then applies the after function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    /**
     * Returns a function that always returns its input argument.
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```
在 Function 接口中定义了两个 default 方法：`compose` 和 `andThen` 可以进行链式的调用，假设有两个函数 f(x) 和 g(x):
```
f.compose(g) => f(g(x))
f.andThen(g) => g(f(x))
```

下图是一个详细的解释

![](/images/lambda/function.png)


下面是一个使用示例：
```java
public void testFunction() {
    Function<Integer, Integer> f = x -> x + 1;
    Function<Integer, Integer> g = x -> x * 2;
    int i = func(1, f.andThen(g));
    // 输出 4
    System.out.println(i);

    i = func(1, f.compose(g));
    // 输出 3
    System.out.println(i);
}
```

#### UnaryOperator
UnaryOperator 是一种特殊的 Function，表示操作数和返回值是同一种类型，函数描述符为 `T -> T`。下面是该接口的定义：
```java
/**
 * Represents an operation on a single operand that produces a result of the
 * same type as its operand.  This is a specialization of {@code Function} for
 * the case where the operand and result are of the same type.
 */
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {

    /**
     * Returns a unary operator that always returns its input argument.
     */
    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```

下面是一个使用示例：
```java
public void testUnaryOperator() {
    UnaryOperator<Integer> u = x -> x + 1;
    System.out.println(u.apply(1));
}
```

### BiFunction
BiFunction 接收两个参数（T, U），返回一个结果（R），类似于 z=f(x, y)，对应的函数描述符为 `(T, U) -> R`。下面是该接口的具体实现：
```java
/**
 * Represents a function that accepts two arguments and produces a result.
 * This is the two-arity specialization of Function.
 */
@FunctionalInterface
public interface BiFunction<T, U, R> {

    /**
     * Applies this function to the given arguments.
     *
     * @param t the first function argument
     * @param u the second function argument
     * @return the function result
     */
    R apply(T t, U u);


    /**
     * Returns a composed function that first applies this function to
     * its input, and then applies the {@code after} function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     */
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
}
```

下面是一个使用示例：
```java
public void testBiFunction() {
    BiFunction<Integer, Double, String> b = (i, d) -> String.valueOf(i + d);
    String r = b.apply(1, 2.5);
    System.out.println(r);
}
```

#### BinaryOperator
BinaryOperator 是一种特殊的 BiFunction，表示接收的参数和返回的结果都是同一种类型 T，函数描述符为 `(T, T) -> T`。下面是该接口的定义：
```java
/**
 * Represents an operation upon two operands of the same type, producing a result
 * of the same type as the operands.  This is a specialization of
 * BiFunction for the case where the operands and the result are all of
 * the same type.
 */
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    /**
     * Returns a BinaryOperator which returns the lesser of two elements
     * according to the specified Comparator.
     */
    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    /**
     * Returns a  BinaryOperator which returns the greater of two elements
     * according to the specified Comparator.
     */
    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```
下面是一个使用示例:
```java
public void testBinaryOperator() {
    BinaryOperator<Integer> b = (x, y) -> x + y;
    int z = b.apply(1, 3);
    System.out.println(z);

    BinaryOperator<Integer> min = BinaryOperator.minBy((x, y) -> x - y);
    // 输出 1
    z = min.apply(1, 3);
    System.out.println(z);

    // 输出 3
    BinaryOperator<Integer> max = BinaryOperator.maxBy((x, y) -> x - y);
    z = max.apply(1, 3);
    System.out.println(z);
}
```

### Primitive specializations
在上面提到的接口中，都是接受泛型参数，我们知道泛型参数只能是引用类型，也就是说对于 int 这样的基本类型，我们要首先装箱（boxing）成 Integer 类型，在使用的时候再拆箱（unboxing）成 int。虽然 Java 提供了自动装箱机制，但是在性能方面是要付出代价的。所以对于上述的函数式接口，Java 8 提供了针对基本类型的版本，以此来避免输入输出是基本类型时的自动装箱操作。以 Predicate 为例，假设我们要检测一个 int 是否满足某个条件，我们可以使用 IntPredicate ：
```java
public void testIntPredicate() {
    IntPredicate ip = x -> x > 3;
    boolean r = ip.test(4);
    System.out.println(r);
}
```
下面是 IntPredicate 的定义，我们可以看到它将泛型 T 改为了基本类型 int。
```java
/**
 * Represents a predicate (boolean-valued function) of one {@code int}-valued
 * argument. This is the {@code int}-consuming primitive type specialization of
 * {@link Predicate}.
 */
@FunctionalInterface
public interface IntPredicate {

    boolean test(int value);

    default IntPredicate and(IntPredicate other) {
        Objects.requireNonNull(other);
        return (value) -> test(value) && other.test(value);
    }

    default IntPredicate negate() {
        return (value) -> !test(value);
    }

    default IntPredicate or(IntPredicate other) {
        Objects.requireNonNull(other);
        return (value) -> test(value) || other.test(value);
    }
}
```
下表列出了 Java 8 中的函数式接口以及其对应的基本类型版本：

| 函数式接口 | 函数描述符 | 基本类型版本 |
|-----|-----|:---|
| `Predicate<T>` | `T -> boolean` | `IntPredicate, LongPredicate, DoublePredicate` |
| `BiPredicate<T>` | `(L, R) -> boolean` | |
| `Consumer<T>` | `T -> void` | `IntConsumer, LongConsumer, DoubleConsumer` |
| `BiConsumer<T, U>` | `(T, U) -> void` | `ObjIntConsumer<T>, ObjLongConsumer<T>, ObjDoubleConsumer<T>` |
| `Supplier<T>` | `() -> T` | `BooleanSupplier, IntSupplier, LongSupplier, DoubleSupplier` |
| `Function<T, R>` | `T -> R` | `IntFunction<R>, IntToDoubleFunction, IntToLongFunction, LongFunction<R>, LongToDoubleFunction, LongToIntFunction, DoubleFunction<R>, ToIntFunction<T>, ToDoubleFunction<T>, ToLongFunction<T>` |
| `UnaryOperator<T>` | `T -> T` | `IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator` |
| `BiFunction<T, U, R>` | `(T, U) -> R` | `ToIntBiFunction<T, U>`, `ToLongBiFunction<T, U>`, `ToDoubleBiFunction<T, U>` |
| `BinaryOperator<T>` |  `(T, T) -> T` | `IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator` |
