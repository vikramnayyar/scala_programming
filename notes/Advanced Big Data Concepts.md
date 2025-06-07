
## 1. Traits (Parent Classes) & Mixins
* Traits --understand_as--> Parent Class
**Why:** Reuse common analytics behavior, e.g., logging, data cleaning.

```scala
// Parent Class 1 - called Trait
trait Logger {
  def log(msg: String): Unit = println(s"[LOG] $msg")      // Unit --means--> void, if we dont want any return from function
}

// Parent Class 2 - called Trait
trait DataCleaner {
  def cleanPromoData(data: Seq[String]): Seq[String] = data.filter(_.nonEmpty)      // Seq[String] is list of strings, --can_be--> List or Array (any) 
}                

// 3. Child Class 1
class PromoAnalytics extends Logger with DataCleaner {
  def analyze(data: Seq[String]): Unit = {                 // 1. Seq[String] ----> pd.Series  2. Unit means return void
    log("Starting analysis")
    val cleanData = cleanPromoData(data)
    log(s"Cleaned data count: ${cleanData.size}")      
    // perform analysis
  }
}
```

* **Seq[String] Example Declaration**
```
val list1: Seq[String] = Seq("a", "b", "c")      //Seq[String] --same_as--> pd.Series --say--> df['product_id']
```

*Nugget:* Use traits to compose reusable analytics modules (cleaning, validation, logging).

* **Optional-Trait vs Mixin**
| Trait Content                                 | Role                          | Used as Mixin?                                  |
| --------------------------------------------- | ----------------------------- | ----------------------------------------------- |
| Has one or more concrete methods (functions)  | Adds behavior                 | ✅ Yes, it's a mixin                             |
| Has only abstract method signatures (no code) | Defines an interface          | ⚠️ Maybe, often a mixin if combined with others |
| Has only constant values (vals)               | Data holder, config container | ❌ Usually *not* a mixin                         |
 

---
## 2. Using Futures for Concurrencies

**Why:** Parallelize promo data fetch or Spark job orchestration.

* Need for Futures & Concurrency

| Step                          | How async is handled?                                    |
| ----------------------------- | -------------------------------------------------------- |
| Reading/writing data in Spark | Spark parallelizes internally — no explicit async needed |
| Running multiple Spark jobs   | Use Futures or async to run jobs concurrently            |
| External API calls in job     | Use async/Futures to avoid blocking                      |
| Orchestration outside Spark   | Use async to parallelize control flows                   |


* Know Futures

| Term         | Definition                             | Example (What happens)                                      |
| ------------ | -------------------------------------- | ----------------------------------------------------------- |
| Synchronous  | Tasks run one-by-one, wait for each    | Program waits 2 sec, then task finishes                     |
| Asynchronous | Tasks run without waiting, can overlap | Program starts task, does other things, task finishes later |
| Future       | Placeholder for a result not ready yet | Future holds a promise for a value coming later             |


* Use Futures --for--> Concurrencies (Parallel Jobs)

* Applications
| Use Case                                    | Why Concurrency Helps                                                |
| ------------------------------------------- | -------------------------------------------------------------------- |
| **Reading multiple datasets**               | Parallel reads speed up ingestion                                    |
| **Running multiple independent Spark jobs** | Run jobs for different pipelines, stores, or datasets simultaneously |
| **Calling external APIs/services**          | Avoid waiting/blocking on slow network calls                         |
| **Complex workflows orchestration**         | Run independent tasks or stages concurrently to save time            |
| **Data processing & transformations**       | Concurrent steps or iterative jobs                                   |
| **Writing to multiple sinks**               | Parallel writes reduce overall pipeline time                         |



```scala
import scala.concurrent._
import ExecutionContext.Implicits.global
import scala.util.{Success, Failure}

def fetchPromoData(storeId: Int): Future[Seq[String]] = Future {
  // simulate data fetch delay
  Thread.sleep(1000)
  Seq("promoA", "promoB")
}

val futureData = fetchPromoData(101)

futureData.onComplete {
  case Success(data) => println(s"Received promo data: $data")
  case Failure(e) => println(s"Fetch failed: ${e.getMessage}")
}
```

*Nugget:* Use `Future` to asynchronously load or process large data chunks without blocking.

---

## 3. Implicit Parameters & Conversions

**Why:** Pass common configs implicitly (e.g., date format, thresholds).

```scala
implicit val defaultThreshold: Double = 0.1

def filterPromosByLift(lifts: Seq[Double])(implicit threshold: Double) =
  lifts.filter(_ >= threshold)

val promoLifts = Seq(0.05, 0.2, 0.15)
println(filterPromosByLift(promoLifts)) // uses implicit threshold 0.1
```

*Nugget:* Simplify function calls by providing analytics parameters implicitly.

---

## 4. For-Comprehensions

**Why:** Chain transformations on options, futures, or datasets cleanly.

```scala
val maybeLift: Option[Double] = Some(0.25)
val maybeBaseline: Option[Double] = Some(1000)

val maybePromoImpact = for {
  lift <- maybeLift
  baseline <- maybeBaseline
} yield lift * baseline

println(maybePromoImpact)  // Some(250.0)
```

*Nugget:* Compose multiple optional or async analytics steps elegantly.

---

## 5. Error Handling: Try & Either

**Why:** Safely handle bad or missing promo data.

```scala
import scala.util.{Try, Success, Failure}

def parseLift(str: String): Try[Double] = Try(str.toDouble)

val liftTry = parseLift("0.15")
liftTry match {
  case Success(value) => println(s"Lift: $value")
  case Failure(_) => println("Invalid lift value")
}
```

Or with `Either` for better error messages:

```scala
def parseLiftEither(str: String): Either[String, Double] =
  try Right(str.toDouble)
  catch { case _: NumberFormatException => Left("Invalid number") }

parseLiftEither("abc") match {
  case Right(value) => println(s"Lift: $value")
  case Left(error) => println(error)
}
```

*Nugget:* Use `Try` or `Either` to gracefully handle data parsing and validation errors.

---

## 6. Collections: Mutable vs Immutable

**Why:** Prefer immutable for safe Spark transformations; mutable for local cache.

| Collection Type | Mutable? | Use Case in Promo Analytics                                                    |
| --------------- | -------- | ------------------------------------------------------------------------------ |
| `List`          | No       | Immutable sequences of promo events                                            |
| `Array`         | Yes      | Fixed-size mutable arrays for temp data                                        |
| `ArrayBuffer`   | Yes      | Dynamic collections during data prep                                           |
| `Vector`        | No       | Efficient immutable random access                                              |
| `Map`           | Both     | Key promo data lookups (mutable for local cache, immutable in transformations) |
| `Set`           | Both     | Unique promo IDs, checking duplicates                                          |

Example:

```scala
val promoIds: List[Int] = List(101, 102, 103)  // immutable
val promoLiftMap = scala.collection.mutable.Map[Int, Double]()
promoLiftMap += (101 -> 0.15)
```

*Nugget:* Immutable collections prevent bugs in distributed data pipelines; mutable help in single-node data prep.

---

### Bonus: Quick Spark integration example

```scala
import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.functions._

val spark = SparkSession.builder.appName("CPGPromo").getOrCreate()

val promoDF = spark.read.option("header", "true").csv("promo_data.csv")

// Using for-comprehension style with DataFrame API
val filteredDF = promoDF.filter(col("lift") > 0.1)

filteredDF.show()
```

