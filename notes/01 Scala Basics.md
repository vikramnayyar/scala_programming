
### 1. **Scala Syntax and Basics**

* **Variables**

  ```scala
  var x: Int = 5       // mutable - 
  val x: Int = 5       // immutable
  val y: String = "Hi" // immutable ----> more computationally efficient
  ```
* **val vs var --only_for--> Overwriting Control** 
```
val listVal = List(1, 2, 3)
// listVal = List(4, 5)  // Error: reassignment to val

var listVar = List(1, 2, 3)
listVar = List(4, 5)     // Allowed: variable reassigned

// Neither listVal nor listVar can do element mutation:
listVal(0) = 10         // Error: value update not allowed
listVar(0) = 10         // Error: value update not allowed
```

* **Data Types**
  `Int`, `Double`, `Boolean`, `String`, `Char`, `Unit` (like void), `Any`, `Nothing`

* **Functions**
  * No Returns ----> Last Statement --gets--> Returned (Always) 
  ```scala
  def add(a: Int, b: Int): Int = a + b        

  // Anonymous function (lambda)
  val multiply = (x: Int, y: Int) => x * y
  ```

* **Control Structures**
  ```scala
  if (x > 0) println("Positive") else println("Non-positive")

  for (i <- 1 to 5) println(i)

  while (x < 10) { x += 1 }
  ```

  ```
  def describeNumber(n: Int): String = {
    if (n < 0)
      "negative"
    else if (n == 0)
      "zero"
    else
      "positive"  // This is the last expression — it will be returned
  }
  ```

  * Return Works ----> Not Used --for--> Convinience | Simplicity
  ```
  def describeNumber(n: Int): String = {
    if (n < 0)
      return "negative"
    else if (n == 0)      // double equal is same in Scala also
      return "zero"
    else if (n > 0)      // double equal is same in Scala also
      return "positive"  
  }
  ``` 
  * Above code --gives_error--> Must Include ----> n>0 (All if cases, Unlike Python)

  * Braces ----> Needed ---->  Multiline Loops --not_needed--> Single Loops
  ```
  if (n < 0) {
  println("Negative number")
  "negative"
  }
  ```

  * Calling Function
  ```
  def add(a: Int, b: Int): Int = {
    val sum = a + b
    sum
  }
  
  val x = 1
  val y = 2
  
  val res = add(x, y)
  println(res)  // This will print 3
  ```
  
### 2. **Object-Oriented & Functional Features**

* **Classes and Objects**

```
def add(a: Int, b: Int): Int = a + b

// Anonymous function
val multiply = (x: Int, y) => x * y

// below is var not function
val multiply = (x: Int, y) => x * y
```

* **Traits** (like interfaces)
```scala
trait Logger {
  def log(message: String): Unit = println(s"LOG MESSAGE: $message")
}

class UserService extends Logger {
  def createUser(name: String): Unit = {
    log(s"Creating user: $name")
    // user creation logic here...
  }
}

class OrderService extends Logger {
  def placeOrder(orderId: Int): Unit = {        // remember to stay in same line, unlike Python
    log(s"Placing order: $orderId")
    // order placement logic here...
  }
}

object Main extends App {                  // Scala automatically creates Main class that extends from App ----> App is inbuilt in Scala (and widely used for convinience)
  val userService = new UserService()
  userService.createUser("Alice")

  val orderService = new OrderService()
  orderService.placeOrder(12345)
}
```

* **Functions as First-Class Citizens**
  Pass functions as parameters, return functions.


### 3. **Collections & Operations**

* **Collections**

  ```scala
  val list = List(1, 2, 3)
  val array = Array("a", "b", "c")
  val map = Map("a" -> 1, "b" -> 2)
  val set = Set(1, 2, 3)
  ```

* **List vs Array**
  * Lists ----> Immutable --can--> Prepend | Append
  ```
  val list = List(1, 2, 3)
  val newList = 0 :: list  // Prepend 0, returns new list: List(0, 1, 2, 3)
  println(list)            // Original list unchanged: List(1, 2, 3)
  println(newList)         // New list: List(0, 1, 2, 3)
  ```

  * Array ----> mutable --cannot--> append | prepend     
  ```
  val array = Array(1, 2, 3)
  array(0) = 10           // Update element at index 0
  println(array.mkString(", "))  // Prints: 10, 2, 3
  
  ```

  * Operations in Lists
  zipWithIndex --same_as--> enumerate()
  zipWithIndex --works--> Lists + Arrays (any sequences)

  ```
  val list = List("a", "b", "c")
  
  for ((value, index) <- list.zipWithIndex) {
    println(s"Index: $index, Value: $value")
  }
  ```

  * Common Operations on Data Structures
  
  | Operation            | List                            | Array                           | Set                             |
  | -------------------- | ------------------------------- | ------------------------------- | ------------------------------- |
  | **Create**           | `List(1, 2, 3)`                 | `Array(1, 2, 3)`                | `Set(1, 2, 3)`                  |
  | **Access by index**  | `list(i)` (slow, O(n))          | `array(i)` (fast, O(1))         | No direct indexing (unordered)  |
  | **Iteration**        | `list.foreach(println)`         | `array.foreach(println)`        | `set.foreach(println)`          |
  | **Append**           | `list :+ 4` (slow, creates new) | Not possible (fixed size)       | `set + elem` (creates new set)  |
  | **Prepend**          | `elem :: list` (fast)           | Not applicable                  | Not applicable                  |
  | **Update element**   | Not possible (immutable)        | `array(i) = newValue` (mutable) | Not applicable (immutable)      |
  | **Filter**           | `list.filter(_ > 2)`            | `array.filter(_ > 2)`           | `set.filter(_ > 2)`             |
  | **Map/Transform**    | `list.map(_ * 2)`               | `array.map(_ * 2)`              | `set.map(_ * 2)`                |
  | **Contains**         | `list.contains(3)`              | `array.contains(3)`             | `set.contains(3)` (fast lookup) |
  | **Length**           | `list.length`                   | `array.length`                  | `set.size`                      |
  | **Zip with index**   | `list.zipWithIndex`             | `array.zipWithIndex`            | No natural order, not typical   |
  | **Convert to other** | `list.toArray`, `list.toSet`    | `array.toList`, `array.toSet`   | `set.toList`, `set.toArray`     |

  * Case --always_replaceable--> if-else Ladder --can_stay--> Pythonic


  * Set Operations

| Operation              | Syntax                          | Description                          | Example                                       | Result                      |                |
| ---------------------- | ------------------------------- | ------------------------------------ | --------------------------------------------- | --------------------------- | -------------- |
| Create a set           | `val s = Set(1, 2, 3)`          | Immutable set of elements            | `Set(1, 2, 3)`                                | `Set(1, 2, 3)`              |                |
| Add element            | `s + 4`                         | Returns new set with element added   | `Set(1, 2, 3) + 4`                            | `Set(1, 2, 3, 4)`           |                |
| Remove element         | `s - 2`                         | Returns new set with element removed | `Set(1, 2, 3) - 2`                            | `Set(1, 3)`                 |                |
| Union                  | `s1 union s2` or \`s1           | s2\`                                 | Combine elements from both sets               | `Set(1, 2) union Set(2, 3)` | `Set(1, 2, 3)` |
| Intersection           | `s1 intersect s2` or `s1 & s2`  | Elements common to both sets         | `Set(1, 2, 3) intersect Set(2, 3, 4)`         | `Set(2, 3)`                 |                |
| Difference             | `s1 diff s2` or `s1 &~ s2`      | Elements in s1 but not in s2         | `Set(1, 2, 3) diff Set(2, 4)`                 | `Set(1, 3)`                 |                |
| Check contains element | `s.contains(elem)` or `s(elem)` | Check if element exists              | `Set(1, 2, 3).contains(2)` or `Set(1,2,3)(2)` | `true`                      |                |
| Size                   | `s.size`                        | Number of elements in the set        | `Set(1, 2, 3).size`                           | `3`                         |                |


* **Numerical Ooperations**

| Operation   | Scala Syntax                    | Returns | Notes                             |
| ----------- | ------------------------------- | ------- | --------------------------------- |
| **Sum**     | `list.sum`                      | `15`    | Works directly on numeric lists   |
| **Max**     | `list.max`                      | `5`     | Largest element                   |
| **Min**     | `list.min`                      | `1`     | Smallest element                  |
| **Average** | `list.sum.toDouble / list.size` | `3.0`   | Manual mean (no built-in `.mean`) |
| **Length**  | `list.length`                   | `5`     | Number of elements                |


* **Statistical Operations - Breeze**
```
import breeze.stats._

val data = DenseVector(1.0, 2.0, 3.0, 4.0, 5.0)    // DenseVector --just-->  np.array --limited--> Slicing | Broadcasting

mean(data)     // 3.0
median(data)   // 3.0
variance(data) // 2.0
```

| Metric       | Native Scala Support | With Breeze (Recommended) | Manual Computation Possible |
| ------------ | -------------------- | ------------------------- | --------------------------- |
| **Mean**     | ✅ (manual)           | ✅ `mean()`                | ✅                           |
| **Median**   | ✅ (manual)           | ✅ `median()`              | ✅                           |
| **IQR**      | ❌                    | ❌ (manual needed)         | ✅                           |
| **Skewness** | ❌                    | ✅ `skewness()` (Breeze)   | ✅ (complex)                 |
| **Kurtosis** | ❌                    | ✅ `kurtosis()` (Breeze)   | ✅ (complex)                 |
| **Std Dev**  | ❌                    | ✅ `stddev()`              | ✅                           |


    

### 4. **Immutability & Case Classes**

* **Immutability**
  Use `val` to declare immutable values. Collections are immutable by default (`List`, `Map`).

* **Case Classes**
  Useful for immutable data structures with built-in methods.

  ```scala
  case class Person(name: String, age: Int)

  val p1 = Person("Bob", 30)
  println(p1.name)      // Bob
  ```


### 5. **Pattern Matching & Option**

* **Pattern Matching**

  ```scala
  val x = 5
  x match {
    case 1 => println("One")
    case 2 | 3 => println("Two or Three")
    case _ => println("Other")
  }
  ```

* **Option (Safe Null Handling)**

  ```scala
  val maybeInt: Option[Int] = Some(10)
  val noneInt: Option[Int] = None

  maybeInt match {
    case Some(value) => println(value)
    case None => println("No value")
  }

  // or using getOrElse
  println(maybeInt.getOrElse(0))  // 10
  ```

---

### Recommended Resources

* **Scala Official Documentation:** [https://docs.scala-lang.org/](https://docs.scala-lang.org/)
* **Books:** *Programming in Scala* by Odersky, Spoon, Venners
* **Interactive Tutorials:** [https://www.scala-exercises.org/](https://www.scala-exercises.org/)

---






| **Level**                          | **Description**                                                          | **Key Metrics / Checks**                                                                                        | **L1**                                      | **L2**                                                                               |
| ---------------------------------- | ------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- | ------------------------------------------- | ------------------------------------------------------------------------------------ |
| **1. Completeness**                | Ensure all required data is present and no records or fields are missing | % missing values per feature<br>Batch row count consistency<br>Missing field detection                          | Missing values<br>Row counts                | Null rate<br>Missing feature flags<br>Empty strings<br>Non-null % per column         |
| **2. Consistency**                 | Verify data adheres to expected schemas, formats, and business rules     | Data type validation<br>Format checks (dates, codes)<br>Referential integrity                                   | Schema checks<br>Business rules             | Data type match<br>Pattern mismatch<br>Foreign key match rate<br>Value format errors |
| **3. Uniqueness & Cardinality**    | Confirm data uniqueness and valid categorical cardinality                | Duplicate record detection<br>Cardinality checks on categorical features                                        | Uniqueness checks<br>High-cardinality flags | Duplicate rows<br>Repeated IDs<br>Cardinality ratio<br>Unexpected new categories     |
| **4. Drift Detection**             | Detect distributional changes compared to baseline or historical data    | PSI, KS-test p-values<br>Shifts in mean, median, skewness, kurtosis                                             | KS test<br>PSI                              | Skewness<br>Kurtosis<br>Mean shift<br>Median shift<br>Distribution overlap           |
| **5. Stability Over Time**         | Track feature value stability and variability trends across batches      | Variance/standard deviation trends<br>Feature value trend analysis<br>Sample size stability                     | Levene test<br>Trend analysis               | Std dev<br>Mean/median fluctuation<br>Week-over-week delta<br>Seasonal variance      |
| **6. Anomaly & Outlier Detection** | Identify rare or unexpected values that may affect training              | Outlier counts and magnitude<br>Sudden spikes in nulls or zeros<br>Unusual correlations or feature interactions | Z-score / IQR<br>Univariate outlier check   | Max/min thresholds<br>Median deviation<br>Rare category frequency<br>Spike detection |
| **7. Data Freshness & Timeliness** | Ensure data is delivered on time and is recent enough for training       | Batch arrival time<br>Data timestamp validation<br>Lag or stale data detection                                  | Timeliness check<br>Latency thresholds      | Delay in arrival<br>Outdated timestamps<br>Time gap from source<br>Late batch alerts |

