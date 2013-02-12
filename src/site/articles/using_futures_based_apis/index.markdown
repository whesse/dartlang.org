--- 
layout: default
title: "Using Futures Based APIs"
description: "A first look at Futures and how to use them in your code."
rel:
    author: shailen-tuli
has-permalinks: true
article:
  written_on: 2013-02-07
  collection: libraries-and-apis
---

# {{ page.title }}
_Written by Shailen Tuli, February 2013_

## Contents

1. [Introduction](#introduction)
1. [What is a Future](#introduction)
1. [Using a Future](#using-a-future)
1. [Sequence of events in asynchronous code execution](#sequence-of-events-in-asynchronous-code-execution)
1. [Handling errors when dealing with Futures](#handling-errors-when-dealing-with-futures)
1. [Chaining functions with futures](#chaining-functions-with-futures)
1. [Waiting on multipel futures to complete](#waiting-on-multiple-futures-to-complete)
{:.toc}

## Introduction

Dart is a single-threaded programming language. This means that if any code
blocks the thread of execution, the program effectively freezes. Let's look at
some code where this happens:

{% prettify dart %}
import 'dart:io';

void printDailyNewsDigest() {
  File file = new File("dailyNewsDigest.doc");
  print(file.readAsStringSync());
}

void main() {
  printDailyNewsDigest();
  printWinningLotteryNumbers();
  printWeatherForecast();
  printBaseballScore();
}
{% endprettify %}

Our program reads the news of the day from a file, `dailyNewsDigest.doc`,
prints it, and then prints a bunch of other items of interest to the user:

    <Contents of dailyNewsDigest.doc>
    Winning lotto numbers: [23, 63, 87, 26, 2]
    Tomorrow's forecast: 70F, sunny.
    Baseball score: Red Sox 10, Yankees 0

While the code is conceptually simple, it is problematic:
since `readAsStringSync()` blocks, the remaining code runs only when
`readAsStringSync()` gets done, _however long that takes_.  And if it does
take long, the user waits passively, wondering if she won the lottery, what
tomorrow's weather will be like, and who won today's game. Not good.

To avoid situations like these, Dart library authors use an asynchronous
model when defining functions that do potentially expensive work.  Such
functions return their value using a Future.

## What is a Future?

A Future represents a means for accessing a value which is available sometime
in the future. A function that returns a Future:

1. Queues up work to be done.
1. Returns a Future object immediately.
1. Later, when a value is available, completes the Future object
with that value (or with an error; we'll discuss that later).

Receivers of a future can register callbacks that handle the value when it
becomes available. 

## Using a Future

Let's rewrite `printDailyNewsDigest()` to read the contents of the news file
asynchronously:

{% prettify dart %}
import 'dart:io';
import 'dart:async';

void printDailyNewsDigest() {
  File file = new File("dailyNewsDigest.doc");
  Future future = file.readAsString();
  future.then((content) {
    print(content);
  });
}
{% endprettify %}

The `printDailyNewsDigest()` function now uses `readAsString()`, which is
non-blocking. Calling `readAsString()` queues up the work to be done but
doesn't stop the rest of the code from executing. The program prints the
lottery numbers, the forecast, and the baseball score; when
`readAsString()` finishes reading the news file, the program prints its
contents. If `readAsString()` takes a little while to complete its work, no
great harm is done: the user gets to read other things before the daily news
digest is printed.

    Winning lotto numbers: [23, 63, 87, 26, 2]
    Tomorrow's forecast: 70F, sunny.
    Baseball score: Red Sox 10, Yankees 0
    <Contents of dailyNewsDigest.doc>

## Sequence of events in asynchronous code execution

The code executes in three simple steps:

1. The program enters `main()`, which calls `printDailyNewsDigest()` and then 
calls the other print functions. After that, `main()` exits; the program continues.
1. Calling `printDailyNewsDigest()` invokes the `readAsString()` method.
This method queues up the file-reading task and returns a future. This future
completes when the file contents are available.
1. The callback registered within `then()` fires and prints the contents
of the news file.

Calling `then()` returns a new Future, which completes with the value
returned by `then()`'s callback. This means that calls to `then()` can be
chained (we'll see examples of this later). 

## Handling errors when dealing with Futures

If a future-returning function completes with an error, `then()` also
completes with an error. We can capture that error using `catchError()`:

{% prettify dart %}

void printDailyNewsDigest() {
  File file = new File("dailyNewsDigest.doc");
  Future future = file.readAsString();
  future.then((content) {
    print(content);
  })
  .catchError((e) {
    print(e.error);
  });
}
{% endprettify %}

If `dailyNewsDigest.doc` does not exist, or if it is unavailable for reading,
the code above executes as follows:

1. `readAsString()`'s future completes with an error.
1. `then()`'s future completes with an error.
1.  `catchError()`'s callback handles the error and its future
completes normally; the error does not propagate.

<aside class="alert alert-info" markdown="1">
  Chaining catchError() to then() is a common pattern when working with
  functions that return Futures.
  <strong>
    Consider this pairing the asynchronous equivalent of a try-catch.
  </strong>
</aside>

Like `then()`, `catchError()` returns a new future which completes with
the return value of its callback.

## Chaining functions with Futures

Consider 3 functions,  `expensiveA()`, `expensiveB()` and `expensiveC()` that 
return Futures.  You can invoke them sequentially (one function starts when a
previous one completes), or you can kick-off all of them at the same time and
do something with the all the values returned. The Future interface is fluid
enough to deal with all these use cases.

When invoking these functions in order, it may be tempting to use nested
callbacks:

{% prettify dart %}
expensiveA().then((aValue) {
  ...
  expensiveB().then((bValue) {
    ...
    expensiveC().then((cValue) {
      ...
    });
  });
});
{% endprettify %}

This works, but nested code can be hard to read, especially if the nesting is
very deep. 

As a cleaner alternate to nesting, you should consider chaining the
functions using multiple `then()` calls:

{% prettify dart %}
expensiveA()
.then((aValue) => expensiveB()) 
.then((bValue) => expensiveC()) 
.then((cValue) => doSomethingWith(cValue));
{% endprettify %}

The net result is the same as when the functions were invoked using nested
callbacks, but the code is simpler to grock.

## Waiting on multiple futures to complete

If the order of execution of the three functions is not important, 
you can use `Future.wait()` to handle multiple Future objects
without having to explicitly chain function calls.

This way, the functions get triggered in quick succession; when all of them
complete with a value, `Future.wait()` returns a new future.
This future is completed with a list consisting of the values produced by
each function.

{% prettify dart %}
Future.wait([expensiveA(), expensiveB(), expensiveC()]).then((valueList) {
  // All the returned values are collected inside valueList.
});
{% endprettify %}

If any of the invoked functions completes with an error, the future returned
by `Future.wait()` also completes with an error.
