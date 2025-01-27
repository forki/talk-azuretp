- title : Taming Types in the Cloud
- description : Azure and F# with Type Providers
- author : Isaac Abraham
- theme : night
- transition : default

# Taming Types in the Cloud

***

## About me

---

### I'm Isaac Abraham!

![](images/isaac.jpg)

---

### Other metadata...

* Working with .NET since 1.0
* Using F# for ~5 years
* Microsoft MVP
* Based in Fulda, Germany (and occasionally London, UK)
* Founder of Compositional IT

<br><br><br>

<a href="https://compositional-it.com">
    <img src="images/CIT-Circle.png" style="width: 200px;"/>
</a>

---

### I also make infamous PRs...

![](images/nugate.png)

---

### What about you?

***

![](images/fsharp256.png)

---

* General purpose programming language
* Functional-*first*
* Powerful type system
* Awesome data manipulation capabilities
* Leads to the "pit of success"

---

![](images/POS-1.png)

---

![](images/POS-2.png)

---

| C# / VB .NET | F#
|-:|:-
| Mutable by default | Immutable by default
| Side-effects and statements | Expressions
| Classes | Functions as values
| Inheritance | Composition
| State | Separate data & functions
| Polymorphism | Algebraic Data Types

---

## F# Primer in < 5 minutes

---

### Values

```fsharp
// bind 5 to x
let x:int = 5

// type inference
let inferredX = 5

// functions are just values, don't need a class
let helloWorld(name) = sprintf "Hello, %s" name 

// type inference again
let text = helloWorld "isaac"
```

---

### Types

```fsharp
open System

// Tuples are first class citizens in F#
let person = Tuple.Create("Isaac", 37)
let personShortHand = "isaac", 37 // string * int
let name, age = personShortHand // decompose the tuple 

// Declaring a record
type Person = { Name : string; Age : int }

// Create an instance
let me = { Name = "Isaac"; Age = 37 }
printfn "%s is %d years old" me.Name me.Age
```
---

### More Types

```fsharp
open FSharp.Data.UnitSystems.SI.UnitSymbols

type Direction = North | South | East | West
type Weather =
    | Cold of temperature:float<C>
    | Sunny
    | Wet
    | Windy of Direction * windspeed:float<m/s>

// Create a weather value
let weather = Windy(North, 10.2<m/s>)

let (|Low|Medium|High|) speed =
    if speed > 10.<m/s> then High elif speed > 5<m/s>. then Medium else Low
```
---

### Exhaustive pattern matching

```fsharp
match weather with
| Cold temp when temp < 2.0<C> -> "Really cold!"
| Cold _ | Wet -> "Miserable weather!"
| Sunny -> "Nice weather"
| Windy (North, High) -> "High speed northernly wind!"
| Windy (South, _) -> "Blowing southwards"
| Windy _ -> "It's windy!"
```

---

### Asynchronous support

```fsharp
open System
open System.Net

let webPageSize = async {
    use wc = new WebClient()
    let! result = wc.AsyncDownloadString(Uri "http://www.bbc.co.uk")
    return result.Length }
```

---

### No nulls!
![](images/null.png)

---

### ACHTUNG!

![alt](images/horror.jpg)

---

### Whitespace sensitive 

```fsharp
open System
    
let prettyPrintTime() =
    let time = DateTime.UtcNow
    printfn "It is now %d:%d" time.Hour time.Minute
```
---

### Equals is comparison!

```fsharp
let x = 5
x = 10 // false, COMPARISON!!!
```
---

### Immutable by default

```fsharp
let a = 10
//a <- 20 // not allowed

let mutable y = 10 // need an extra keyword!
y <- 20 // ok 
```

---

### REPL

* Read, Evaluate, Print Loop
* No console applications needed
* Scripts
* Explore domain quickly
* Converts quickly to full-blown assemblies

***

![](images/azure.png)

---

* Lower costs
* Reduce cap ex
* "Scale fast, fail fast" etc. 
* Reduce time to market
* Enable distributed applications

---

* Deploying
* Replication
* Load balancing
* Logging
* Scale up
* Scale out
* Auth
* Resiliency

---

### Azure and F#?

![alt](images/Suspicious.png)

---

### Azure and F#!

![](images/friends.jpg)

---

### F# runs on .NET!

---

<a href="https://docs.microsoft.com/en-us/dotnet/fsharp/using-fsharp-on-azure/">
    <img src="images/azure-docs-1.png" style="width: 700px;"/>
</a>

---

### Now with VS "cool demo mode" support!

![](images/azurepublish.png)

---

| Cloud Applications | F# |
|:---:|:---:|
Stateless | Immutable, Expressions
Data-centric | Type Providers, Pattern Matching
Fault tolerant | Powerful type system
Asynchronous | async { }
Distributed | cloud { }

***

## Data in Azure

---

### Azure Storage

![](images/azure-storage.jpg)

---

### SQL Azure

![](images/azure-sql.png)

---

### And others...

* Cosmos DB
* Redis Cache
* Data Lake
* etc. etc.

---

<img src="images/costs.png" style="width: 600px;"/>

---

### Comparing SQL and Storage

| | SQL Azure | Azure Storage |
|---:|:---:|:---:|
| **Cost** | High | Low |
| **Compute** | Powerful | Dumb |
| **Scalability** | Manual | Automatic |

---

### Compute on Azure data services

| | SQL | Tables | Blobs
---:|:---:|:---:|:---:| 
**Projection** | Yes | Yes | No
**Filters** | Yes | *Limited* | No
**Joins** | Yes | *No* | No
**Relationships** | Yes | *No* | No
**Indexes** | Yes | *Limited* | Limited

---

### Working with SQL Azure

* *Table*-level schema
* Relatively rich type system

Customer Id | Name | Order Count | Balance
| --- | --- | --- | ---|
| *guid* | *string 50 null* | *int* | *decimal* |
| 2542685a-... | Joe Bloggs | 23 | 126.23
| bcf678fb-... | Sally Smith | 12 | 59.10
| ad081c1b-... | ***{null}*** | 17 | 89.23

---

### Working with Azure Tables

* *Row*-level schema
* No max length etc.
* All columns are nullable

| Customer Id | Name | Order Count | Balance |
| --- | --- | --- | --- |
| 2542685a-... | Joe Bloggs | 23 | ***{N/A}***
| **123** | Sally Smith | 12 | 59.10
| ad081c1b-... | ***{N/A}*** | 17 | 89.23

---

```fsharp
{ CustomerId = "2542685a"; Name = "Joe"; OrderCount = 23 }
{ CustomerId = 123; Name = "Sally"; OrderCount = 12; Balance = 59.10 }
{ CustomerId = 123; OrderCount = 12; Balance = 59.10 }
```
---

### Blob Type System

* *No* entity schema
* No notion of rows or columns
* Data stored as raw documents in a path *e.g.*

<br>
<br>

```json
        { "customerId" : "2542685a-",
          "name" : "Joe Bloggs",
          "orderCount" : 23 }
```

***

## DEVELOPERS!

![](images/developers.jpg)

---

## Working with the Azure Storage SDK

---

### Typical tasks:

* What's in my storage account?
* What do the contents of this file look like?
* What's the schema of this table?
* What does the data in my table look like?
* What's currently on a storage queue?

---

## Working with Blobs

---

### Exploration of data?

---

![](images/sdk-1.png)

---

![](images/sdk-2.png)

---

### Is this easy to explore data?

---

### Type safety?

![](images/sdk-3.png)

---

![](images/sdk-4.png)

---

![](images/sdk-5.png)

---

![](images/sdk-6.png)

---

## Working with Tables

---

### Type Safety?

---

![](images/sdk-7.png)

---

![](images/sdk-8.png)

---

![](images/sdk-9.png)

---

![](images/sdk-10.png)

---

![](images/sdk-11.png)

---

### More type safety

![](images/sdk-17.png)

---

### Code like this...

![](images/sdk-18.png)

---

### Leads to messages like this...

![](images/sdk-19.png)

---

### The Pit of Success?

![](images/sdk-12.png)

---

![](images/sdk-13.png)

---

![](images/sdk-14.png)

---

![](images/wtf.jpg)

---

### Impedence mismatches

![](images/sdk-15.png)

---

![](images/sdk-16.png)

---

![](images/sdk-20.png)

---

![](images/wtf.jpg)

---

## Question:
## How can F# improve this?

---

## What is the Storage TP?

---

[<img src="images/sdk-21.png">](http://fsprojects.github.io/AzureStorageTypeProvider/)

---

![](images/bored.jpg)

---

# DEMOS!!

---

### Benefits of F# for data

| | Before | After
|-|:-:|:-:|
| **Missing data** | Null | Option<T> |
| **Error Handling** | Exceptions | Result<T> |
| **Remote resources** | Stringly-typed | Type system |
| **Schema** | Manually generated | Provided |
| **Compile-time Queries** | Unsafe | Safe |

***

### Applications of Storage Type Provider

* Application code for Tables
* Exploration of data
    * Log tables
    * Metrics
    * Exploring "heads" of blobs
    * Exploring unseen tables
* Working with "well-know" blob schemas
* Can always "fall back" to standard Azure SDK

---

## Storage Accounts cost money!

* **Every** query costs
    * **Every** time you dot into something, it costs!
* Use schema files to specify your types
    * Or use preset "dummy data" to guide provided types
* Use the Storage Emulator
    * Repoint to live storage account at runtime

---

## Developing Type Providers

* It's a pain
* It's a REAL pain
* Debugging is next to impossible
* Running 2 IDEs side-by-side
* Slow to develop
* Dependencies are difficult to work with
* .NET Core compatibility **very close**!

---

## Build and CI

![](images/storage-tp-build.png)

---

## Building and Testing

* Tests = compiles :)
    * Need to have the emulator running locally!
* Suite of integration tests on CI
* Appveyor for build
    * Appveyor supports Azure Storage Emulator :)

---

## Shameless plug alert!

<a href="https://www.manning.com/books/get-programming-with-f-sharp">
    <img src="images/get-programming.jpg">
</a>

***

## Thank you!

<a href="https://compositional-it.com">
    <img src="images/CIT-Circle.png" style="width: 200px;"/>
</a>

https://compositional-it.com

[@isaac_abraham](http://twitter.com/isaac_abraham)

[isaac@compositional-it.com](mailto:isaac@compositional-it.com)
