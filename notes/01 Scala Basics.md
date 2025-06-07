
### 1. **Scala Syntax and Basics**

* **Variables**

  ```scala
  var x: Int = 5       // mutable
  val x: Int = 5       // immutable
  val y: String = "Hi" // immutable ----> more computationally efficient
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
  ``
  

### 2. **Object-Oriented & Functional Features**

* **Classes and Objects**

```
def add(a: Int, b: Int): Int = a + b

// Anonymous function
val multiply = (x: Int, y: Int) => x * y
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


### 3. **Collections & Higher-Order Functions**

* **Collections**

  ```scala
  val list = List(1, 2, 3)
  val array = Array("a", "b", "c")
  val map = Map("a" -> 1, "b" -> 2)
  val set = Set(1, 2, 3)
  ```

* **Higher-Order Functions**

  ```scala
  list.map(_ * 2)        // List(2, 4, 6)
  list.filter(_ % 2 == 1) // List(1, 3)
  list.reduce(_ + _)     // 6
  ```

---

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

---

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

