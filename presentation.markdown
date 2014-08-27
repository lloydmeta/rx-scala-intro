title: Intro to RxScala
author:
  name: Lloyd
  twitter: meta_lloyd
  github: lloydmeta
  url: http://beachape.com
output: index.html
theme: sudodoki/reveal-cleaver-theme
controls: true

--

# Intro to RxScala

--

## Overview

_Focus on usage_

1. What is an asynchronous stream?
2. Where are asynchronous streams in a given app?
3. Enter Rx (Reactive Extensions)
4. Rx Observable
5. Rx Observer
6. Schwatcher w/ callbacks
7. Schwatcher w/ Rx
8. Conclusions
9. Resources

--

# What is an
# Asynchronous stream?

--
### What is a collection (immutable)?

A collection is a number of objects defined at 1 instance in time.

* Examples:
  - `Stream.fill(9001)("lo") `<- __not__ asynchronous
  - `Set(1, 2, 3, 4)`
  - `List("a", "b", "c")`
  - `Seq.empty`

--
### _Asynchronous_ stream?

An asynchronous stream is a source of objects, but the objects are produced at different points in time.

![Asynchronous stream marble diagram showing filtering](imgs/filter.png)

--

### Where are asynchronous streams in a given app?

* _Technically_, anywhere you want to handle something asynchronously.

* This means that you can use it to represent a single asynchronous result, such as a `Future`.

* For practical reasons though, similar to how nobody wraps singular objects in `List` or `Seq`, we normally talk about asynchronous streams when there is a possibility of more than 1 result object.

--
### Where are asynchronous streams in a given app ?

## Callbacks.

```javascript
 $("#textbox").addEventListener("keypress", function(event) {
        // do stuff
    });
```

* When we use callbacks like this, the objects (in this case, HTML events) are all ephemeral; they are "lost" after the function is run (unless if we store them manually).

* Things also quickly become unmaintainable if we want to support different behaviour depending on the event (e.g. only run the function if the key was "3")

--

# Enter the Rx

![Rx](imgs/rx_logo.png)

--
## Rx (Reactive Extensions) Framework

* Started off as a .NET library (!) as the asynchronous dual of LINQ (a collection manipulation library). There are now implementations of Rx for various languages.
* Netflix engineering created a port of Rx to Java - [RxJava](https://github.com/ReactiveX/RxJava), and naturally there is a Scala wrapper for RxJava that makes it syntactically nicer to use called [RxScala](https://github.com/ReactiveX/RxScala).

--
## Rx Observable

In Rx, the term _Observable_ is used to describe an asynchronous stream.

So from now on, we will use this (shorter) word.

--

## Observable creation

There are several ways to make an Observable:

```scala
// From a collection
val obsInts: Observable[Int] = Observable.items(1, 2, 3)

// Emission of consecutive values at intervals
val obsLongsAtIntervals: Obervable[Long] = Observable.interval(100 millis)
```

* The RxScala library evolves very quickly so these may already be outdated by the time you see this. Please check [the official examples](https://github.com/ReactiveX/RxScala/tree/master/src/examples/scala/rx/lang/scala/examples) for up-to-date versions.

--
## Callbacks → Observables

_Note_: There are other ways of doing this

```scala
// a Subject is an observable that allows you to put things in it.
val publishSubject = PublishSubject.create[String]
val someSwingTextField = // your Swing TextField
someSwingTextField.subscribe( { case ValueChanged(x) => publishSubject.onNext(x.text) } )
```

* We used a `PublishSubject`, which implements the `Subject` interface AND the `Observable` interface
* `Subjects` allow you to push things into them
* There are different kinds of `Subject`, all of which behave differently. Read about them [here](https://github.com/ReactiveX/RxJava/wiki/Subject)

--

## Composing Obervables

Thanks to RxScala, `Observable` objects have a composition API that is very similar to the standard Scala library.

```scala
val periodic: Observable[Long] = Observable.interval(100 millis) // Start with this
```
--

## Filter

`val onlyOdds = periodic.filter(_ % 2 == 1)`

![filter](imgs/filter.png)

--

## Map

`val times10 = periodic.map(_ * 10)`

![map](imgs/map.png)

--

## Async transformers

There are also other transformers that deal specifically with the async nature of Observables.

--

##Flatmap

![flatMap](imgs/flatMap.png)

--

## Merge

![merge](imgs/merge.png)

--

## Concat

![concat](imgs/concat.png)

--

## ... and many others

For more details, [read the guide](https://github.com/ReactiveX/RxJava/wiki/Transforming-Observables)

--

## Rx Observer

Essentially callback functions as a type.

```scala
trait Observer[-T]  {
  def onNext(value : T) : Unit
  def onError(error : hrowable) : Unit
  def onCompleted() : Unit
}
```

--

## Rx Observer


```scala
val observer = Observer[Int](
  onNext = println(_),
  onError = e => println(e),
  onCompleted = () => println("all done")
)
val subscriptionObserver = intObservable.subscribe(observer)
val subscriptionFunLit = intObservable.subscribe(onNext = println(_))
```

* You pass an Observer to an Observable's `subscribe` method (or a function literal)
* The return type is `Subscription`, which is an object you call `unsubscribe` on to tell the Observable that it should no longer invoke the Observer for that Subscription.

--

## [Schwatcher](https://github.com/lloydmeta/schwatcher)

* File watching library for Scala using Java7+ WatchService API.
* 2 APIs: Callbacks & RxScala:
  * Let's compare the two when it comes to monitoring files in a directory.

--

## Callbacks

```scala
val fileMonitorActor  = // FileMonitor Actor

val printPath = { p: Path => println(s"Something was modified in $p" }

// Callback for 1 file
fileMonitorActor ! RegisterCallback(
  ENTRY_MODIFY
  path = Paths get "/Users/lloyd/Desktop/test" ,
  printPath )

// Callback for another file ...
fileMonitorActor ! RegisterCallback(
  ENTRY_MODIFY
  path = Paths get "/Users/lloyd/Desktop/test2" ,
  printPath )
```

--

## RxScala

```scala
val monitor = RxMonitor()
val observable = monitor.observable

val testObservable = observer.filter(_.path.toString.split('/').last == "test")
val test2Observable = observer.filter(_.path.toString.split('/').last == "test2")

val observer = Observer[EventAtPath](onNext = { event => println(s"Something was modified in: ${event.path}")})

monitor.registerPath(ENTRY_MODIFY, Paths get "/Users/lloyd/Desktop")

testObservable.subscribe(observer)
scalaObservable.subscribe(observer)
```
--

## Conclusion

* Asynchronous streams are a useful abstraction for dealing with asynchronous data in a composable way.
* Rx is an asynchronous stream spec/framework that offers a nice API.

--

## Resources

* Erik Meijer's [Playful intro to Rx video](https://www.youtube.com/watch?v=WKore-AkisY)
* List of Rx implementations on [RafaelF's blog](http://blogs.msdn.com/b/rafaelf/archive/2013/03/18/various-implementations-of-rx.aspx)
* Coursera course on [Reactive Programming](https://www.coursera.org/course/reactive)
* Swatcher file watching lib: [Github](https://github.com/lloydmeta/schwatcher)

--

## Questions?
