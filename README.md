# Java Array and Generics

Arrays know their type at runtime. Java arrays are ***reified***, meaning the JVM tracks the exact element type in memory. When you insert an element, the JVM checks its type and throws an `ArrayStoreException` if it does not match.

```java
public class Main {
  public static void main(String[] args) {
    // Creating and accessing a Simple Array
    Integer[] intPair = new Integer[]{100, 200};
    Integer first = intPair[0];
    System.out.println(first);
  }
}
```

Generics lose their type at runtime. Java implements generics via ***type erasure***, which strips out type parameters (`<T>`) during compilation and replaces them with Object (or their upper bound). Because T no longer exists when the program runs, the JVM cannot know what type of array to allocate in memory for `new T[]`.

* Generic Type: `List<E>`
* Type Parameter: `E`
* Parameterized Types: `List<Integer>`, `List<String>`

```java
import java.util.List;

public class Main {
  public static void main(String[] args) {
    // Creating and accessing a Simple List
    List<Integer> intPair = List.of(100, 200);
    Integer first = intPair.get(0);
    System.out.println(first);
  }
}
```
## Arrays are Covariant

If Class `A` is a subtype of Class `B`, then `A[]` is a subtype of `B[]`.

```java
public class Main {
  public static void main(String[] args) {
    Integer[] pair = {100, 200};
    // treat Integer[] as Number[]
    Number[] nums = pair; // no type casting 
  }
}
```

## Generic Types are Invariant

If Class `A` is a subtype of Class `B`, then `List<A>` is neither a subtype nor a super type of `List<B>`.

```java
import java.util.List;
import java.util.Arrays;

public class Main {
  public static void main(String[] args) {
    List<Integer> list = Arrays.asList(100, 200);

    // Compile error: incompatible types
    List<Number> nums = list;
  }
}
```

## Primary Benefit of Covariance

The elements of an Array can be treated in a generic way as long as we are just reading from the array.

```java
public class Main {
  private static double sum(Number[] nums){
    double result = 0;
    for (Number num : nums){
      result += num.doubleValue();
    }
    return result;
  }

  public static void main(String[] args) {
    // Call sum(..) with Integer[]
    Integer[] ints = {100, 200};
    System.out.println(sum(ints));

    // Call sum(..) with Float[]
    Float[] floats = {3.2F, 5.2F};
    System.out.println(sum(floats)); 
  }
}
```

## Pros and Cons of Invariance

Type incompatibility errors are caught at compile time. 

```java
import java.util.List;

public class Main {
  private static double sum(List<Number> nums){
    double result = 0;
    for (Number num : nums){
      result += num.doubleValue();
    }
    return result;
  }

  public static void main(String[] args) {
    List<Integer> ints = List.of(100, 200);
    // Compile error
    System.out.println(sum(ints));
  }
}
```

## Problem with Covariance
Reduced type safety.

```java
public class Main {
  public static void main(String[] args) {
    Integer[] pair = {100, 200};

    // treat Integer[] as Number[]
    Number[] nums = pair;

    // Set a Float. Will compile.
    nums[0] = 1.2F;

    // Later on - access it as Integer
    Integer first = pair[0];
    System.out.println(first);
  }
}
```

## Solution with Generic Methods

```java
import java.util.List;

public class Main {
  // Handle any List with elements extending from Number
  private static <T extends Number> double sumList(List<T> nums){
    double result = 0;
    for (Number num : nums){
      result += num.doubleValue();
    }
    return result;
  }

  public static void main(String[] args) {
    List<Integer> ints = List.of(100, 200);
    System.out.println(sumList(ints));
  }
}
```

## Solution with Wildcards

```java
import java.util.List;

public class Main {
  // Handle any List with elements extending from Number
  private static double sumList(List<? extends Number> nums){
    double result = 0;
    for (Number num : nums){
      result += num.doubleValue();
    }
    return result;
  }

  public static void main(String[] args) {
    List<Integer> ints = List.of(100, 200);
    System.out.println(sumList(ints));
  }
}
```

## Arrays Type Parameter

```java
public class Main {
  private static <T extends Number> double sum(T[] nums){
    double result = 0;
    for (T num : nums){
      result += num.doubleValue();
    }
    return result;
  }

  public static void main(String[] args) {
    Integer[] intArray = {1, 2, 3};
    System.out.println(sum(intArray));
    Float[] floatArray = {1.1F, 2.2F, 3.3F};
    System.out.println(sum(floatArray));
  }
}
```

Two differences
* Arrays are Covariant, Generic Types are Invariant.
* Arrays enforce their element type at run time (reified); Generic Types enforce their element type at compile time but not run time (due to Type Erasure).

## Covariance vs. Invariance

* Arrays are covariant. If `String` is a subtype of `Object`, then `String[]` is a subtype of `Object[]`. This allows you to assign a specific array to a more general array reference.
* Generics are invariant. `List<String>` is not a subtype of `List<Object>`, preventing you from accidentally mixing incompatible types into a collection.

## Why Combining Them Breaks Type Safety

```java
// Hypothetical illegal code:
List<String>[] stringLists = new List<String>[1]; // Compile-time error!
Object[] objects = stringLists;                   // Legal because arrays are covariant
objects[0] = List.of(42);                         // JVM only sees List[], so ArrayStoreException fails to trigger!
String s = stringLists[0].get(0);                 // Runtime ClassCastException: Integer cannot be cast to String
```

source: [Java Arrays and Generics - Can they Play Together?](https://youtu.be/m1AmbYIJsac)