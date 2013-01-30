--- 
layout: default
title: "Futures and Error Handling"
description: "Learn how to handle errors and exceptions with Futures"
rel:
    author: shailen-tuli
has-permalinks: true
article:
  written_on: 2013-01-27
  collection: libraries-and-apis
---

# {{ page.title }}
_Written by Shailen Tuli, January 2013_

## Introduction
This article covers the basics of error handling when working with Futures.
We will learn about the different ways of dealing with errors and exceptions
in an asynchronous context, explore the nuances of each approach, and attempt
to develop a deeper understanding of how errors propagate through a call stack.

But first, let us begin with a simple example of a function that returns a 
`Future` and show how this function is evoked:

<!-- basics.dart -->

{% highlight dart %}
import 'dart:async';

int theAnswer() {
  return 42;
}

Future<int> expensiveQuery() {
  Completer completer = new Completer();
  new Timer(200, (_) {
    completer.complete(theAnswer());
  });
  return completer.future;
}

void main() {
  expensiveQuery().then((value) {
    print(value); // 42
  });
}
{% endhighlight %}

This works as expected: `expensiveQuery()` returns a `Future`; when the
future completes (approximately 200ms later), its value is accessed in `main()`. 

But what if `theAnswer()` threw an error instead of returning a number?

{% highlight dart %}
int theAnswer() {
  throw new ArgumentError('one, two, three, four'); // an arbitrary error
}
{% endhighlight %}

If we don't make any changes to `expensiveQuery()`, the error is not caught
and goes unhandled in `main()`.

## completeError() and catchError()

Let us now change our code so that the error emanating from `theAnswer()` is 
handled by `expensiveQuery()` and any errors returned by `expensiveQuery()` are
subsequently handled in `main()`:

<!-- simple_error_handling.dart -->

{% highlight dart %}
import 'dart:async';

int theAnswer() {
  throw new ArgumentError('one, two, three, four'); // an arbitrary error
}

Future<int> expensiveQuery() {
  Completer completer = new Completer();
  new Timer(200, (_) {
    int val;
    try {
      val = theAnswer();
      completer.complete(val); // use complete() on success
    } catch(e) {
    completer.completeError(e); // use completeError() on failure
    }
  });
  return completer.future;
}

void main() {
  expensiveQuery().then((value) {
    print('inside then()'); // we don't land here
  }).catchError((e) {
    print('inside catchError()');
    print(e.error); // Illegal argument(s): one, two, three, four
    print(e.runtimeType); // AsyncError
    print(e.error.runtimeType); // ArgumentError
  });
}
{% endhighlight %}

There are a few things to note here:

- we wrap our call to `theAnswer()` in a `try`-`catch`. If `theAnswer()`
returns normally (which it doesn't in its present version), we use `completer.complete`
to return a value; if it `throws`, we use `completer.completeError()` to pass the
error object along to context where `expensiveQuery()` is called (in this case,
`main()`).

- In `main()`, we chain `catchError()` to `then()`. If `expensiveQuery()`
returns normally, the code inside `then()` executes; otherwise, the code
inside `catchError()` executes.

- inside `catchError()`, our original `ArgumentError` is wrapped in
an `AsyncError`. The original `ArgumentError` can be accessed through the 
`error` property of the error object (`e.error.runtimeType`).

In the way we have rewritten `expensiveQuery()`, we get access to the error
object (`e`), but not the stackTrace. To access the stackTrace, pass a second
argument to `completeError()`:

{% highlight dart %}
completer.completeError(AsyncError e, stackTrace);
{% endhighlight %}

The exception and the stackTrace are then combined into an AsyncError and
forwarded to the functions caller. If the original error is an `AsyncError`,
the stackTrace is ignored.

## Registering a pair of callbacks

An alternative to using the chained call to `catchError()` is to register a
pair of callbacks, the first handling success, the second, failure. 
Assuming no change in `expensiveQuery()` and `theAnswer()`, these callbacks
can be registered as follows:

<!-- onError.dart -->

{% highlight dart %}
void main() {
  expensiveQuery().then((value) {
    print('inside then()'); // we don't get here
  }, onError : ((e) {
    print('inside onError');
    print(e.error); // Illegal argument(s): one, two, three, four
  }));
}
{% endhighlight %}

The callback approach (using `onError`) and the `catchError()` chaining both work, but
there is a crucial difference between them: if a new error is thrown inside
`then()`, `onError` does not handle it: 

{% highlight dart %}

int theAnswer() {
  return 42; // we're back to returning a value
}

// expensiveQuery() remains the same as above

void main() {
  expensiveQuery().then((value) {
    print('inside then()');  // we get here...
    throw "new pesky error"; // and we throw a new error    
  }, onError : ((e) {
    print('inside onError'); // we never get here
  }));
}

// Output (stackTrace truncated):
//   inside then()
//   Uncaught Error: new pesky error
//   Stack Trace:
//   ...
//
//   Unhandled exception:
//   new pesky error
//   ...

{% endhighlight %}

`catchError()` does:

{% highlight dart %}
import 'dart:async';

void main() {
  expensiveQuery().then((value) {
    print('inside then()');  // we get here...
    throw "new pesky error"; // and we throw a new error
  }).catchError((e) {
    print('inside catchError()');
    print(e.error); // We don't get the argumentError anymore.
    print(e.error.runtimeType); // this is now String, not ArgumentError
  });
}

// Output:
//   inside then()
//   inside catchError()
//   new pesky error
//   String
{% endhighlight %}

## Handling specific errors
Our error handling has been pretty generic so far: we're catching any errors
that come our way inside `catchError()` (or an `onError` callback). What if we
want to catch a specific error? Or catch more than one error?  

`catchError()` permits the passing of an optional named argument, `test`, that
allows us to query what kind of error was thrown.

    abstract Future catchError(onError(AsyncError asyncError), {bool test(Object error)})

Consider `errorProne()`, a function that capriciously raises errors based
on the argument provided to it:

{% highlight dart %}
Future<int> errorProne(num x) {
  Completer completer = new Completer();
  if (x % 2 == 0) {
    completer.completeError(new ArgumentError()); // arbitrary error
  } else if (x % 3 == 0) {
    completer.completeError(new FallThroughError()); // another arbitrary error
  }
  else {
    completer.complete(42);
  }
  return completer.future;
}
{% endhighlight %}

Here is a way to use `test` to chain the handling of specific errors:

{% highlight dart %}
void main() {
  errorProne(6).then((value) {
    print('inside then');
  }).catchError(((e) {
    print("caught an ArgumentError");
  }), test: (e) => e is ArgumentError)
  .catchError(((e) {
    print("caught an FallThroughError");
  }), test: (e) => e is FallThroughError);
}
{% endhighlight %}

## Handling errors as early as possible

It is crucial that error handlers are installed as early as possible: this avoids 
scenarios where a future is completed with an error, but the error handler is 
not yet attached and the error accidentally propagates. Consider this code:

{% highlight dart %}
void main() {
  Future future = expensiveQuery();
  
  // oops, too late
  new Timer(500, (_) {
    future.then((value) {
      print('inside then()'); // we don't get here
    }).catchError((e) {
      print('inside catchError()'); // we don't get here either
    });
  });
}

// Output:
//   Uncaught Error: Illegal argument(s): one, two, three, four
//   Stack Trace:
//     ...

{% endhighlight %}

`expensiveQuery()` returns after 200ms, but `catchError()` is not registered
until 300ms later and does not catch the error. The problem goes away if 
`expensiveQuery()` is called within the `new Timer()` callback:

<!-- not late -->

{% highlight dart %}
void main() {
  new Timer(500, (_) {
    expensiveQuery().then((value) {
      print('inside then()'); // we don't get here
    }).catchError((e) {
      print('inside catchError()'); // but we do get here this time
    });
  });
}

// Output:
//   inside catchError()

{% endhighlight %}

## immediate() and immediateError()

The async library provides a shortcut for returning a `future` where the
following code:

{% highlight dart %}
Future<int> someFunction() {
  Completer completer = new Completer();
  completer.complete(some_value);
  return completer.future;
}
{% endhighlight %}

can be rewritten more succinctly as:

{% highlight dart %}
Future<int> someFunction() {
  return new Future.immediate(some_value);
}
{% endhighlight %}

So, how can one use this terse syntax and still be able to properly handle
errors and exceptions? Use `immediateError()`. 

{% highlight dart %}
Future<int> someFunction() {
  int val;
  try {
    val = theAnswer();
    return new Future.immediate(val); 
  } catch(e, stacktrace) {
    return new Future.immediateError(AsyncError e); 
  }
}

void main() {
  someFunction().then((value) {
    print(value);
  }).catchError((e) {
    print("error inside someFunction");
  });
{% endhighlight %}

It should be emphasized that `Future.immediate` and `Future.immediateError` should
only be used if the values passed as arguments to them are known immediately. This
is not valid:

{% highlight dart %}
// oops, cannot do this: immediate really does mean immediate, not after 200 ms
Future<int> expensiveQuery() {
  new Timer(200, (_) {
    int val;
    try {
      val = theAnswer();
      return new Future.immediate(val); 
    } catch(e, stacktrace) {
      return new Future.immediateError(e, stacktrace); 
    }
  });
}
{% endhighlight %}

## whenComplete(), the asynchronous equivalent of finally

Sometimes you want some code to run regardless of whether a function that
returns a `Future` actually returned a value or an error, akin to the 
`finally` block in synchronous code.  In such a case, use `whenComplete()`:

{% highlight dart %}
var server = connectToServer();
server.post(myUrl, fields: {"name": "john", "profession": "juggler"})
      .then((response) => handleResponse())
      .catchError(e) => handleError())
      .whenComplete(server.close);
{% endhighlight %}

In this example, we make a call to a server; we want to call `server.close`
regardless of whether we make a successful post request (`.then()`) or
not (`.catchError(()`).  `.whenComplete()` ensures that the connection
is closed in either case.

## Chaining futures

In many of our examples so far, we have called `expensiveQuery()` from
within `main()`. But what if some other function - one that also returns a
`Future` - were to call `expensiveQuery()`? Since `expensiveQuery()` returns 
an error, our new function (let's call is `useExpensiveQuery()`, would have
to somehow process that error and then either return a value or an error. 

Let us write three different versions of `useExpensiveQuery()'.
In the first version, the call to `expensiveQuery()` is handled by a
`.then()`, but no `.catchError()` is used; in the second version, both
`.then()` and `.catchError()` are used and the error from the `expensiveQuery()`
is handled; in the third version, the error is also handled and a new error
is dispatched. 

Consider the first version:

{% highlight dart %}
Future useExpensiveQuery() {
  return expensiveQuery().then((value) {
    print('inside useExpensiveQuery then'); // we don't get here
  });
  // we don't have a catchError()
}

void main() {
  useExpensiveQuery().then((value) {
    print("inside main then");
  }).catchError((e) {
    print("inside main catchError");
  });
}
{% endhighlight %}

The error is not handled in this version, but is passed on. The caller of 
`useExpensiveQuery()` needs to register an appropriate error-handler. If such a
handler is not registered, the error propagates to the global error-handler.

The error _is_ handled in the second version of `useExpensiveQuery()`:
 
{% highlight dart %}
Future useExpensiveQuery() {
  return expensiveQuery().then((value) {
    print('inside useExpensiveQuery then'); // we don't get here 
  }).catchError((e) {
    print('inside useExpensiveQuery catchError'); // we get here
    // The error is handled and not propagated
  });
}

void main() {
  useExpensiveQuery().then((value) {
    print("inside main then"); // we get here, useExpensiveQuery() returned successfully
  }).catchError((e) {
    print("inside main catchError"); // we don't get here; the error was not propagated
  });
}

// Output:
//   inside useExpensiveQuery catchError
//   inside main then

{% endhighlight %}

When `useExpensiveQuery()` is called within `main()`, there is no error to catch. The
error from `expensiveQuery()` was succesfully handled in `useExpensiveQuery()` and
does not propagate.

In the third version of `useExpensiveQuery()`, we also successfully catch the error, but
this time we throw a new exception:

{% highlight dart %}
Future useExpensiveQuery() {
  return expensiveQuery().then((value) {
    print('inside useExpensiveQuery then');    
  }).catchError((e) {
    print('inside useExpensiveQuery catchError');
    // an aribitrary new error
    return new Future.immediateError(new ExpectException("didn't expect this"));
  });
}

void main() {
  useExpensiveQuery().then((value) {
    print("inside main then");
  }).catchError((e) {
    print("inside main catchError");
    print(e.error);
    print(e.runtimeType);
    print(e.error.runtimeType);
  });
}

// Output:
//   inside useExpensiveQuery catchError
//   inside main catchError
//   didn't expect this
//   AsyncError
//   ExpectException
{% endhighlight %}

## Errors in Future.wait()

`Future.wait()` takes a list of futures as an argument and returns a list
of all the values produced by those futures. But if even a single future in the list
completes with an error, the resulting future also completes with an error.

This is easier to understand with a simple example:

{% highlight dart %}
Future<String> pause(num milliseconds) {  
  Completer completer = new Completer();
  
  // simulate expensive activity 
  new Timer(milliseconds, (_) {
    if ((milliseconds % 1000) != 0) {
      completer.completeError(new ArgumentError("only whole seconds, please"));
      return;
    }
    completer.complete("waited $milliseconds milliseconds");
  });
  return completer.future;
}
{% endhighlight %}

`pause()` takes `milliseconds` duration to complete. If `milliseconds` does
not represent a whole second value, it generates an `ArgumentError`.  Calling  
`pause()` repeatedly using `Future.wait()` returns a list of strings when each
argument to `pause()` is a multiple of 1000 (1 second):

{% highlight dart %}
void main() {
  Future.wait([pause(1000), pause(1000), pause(2000)]).then((value) {
    print("inside then");
    print(value);
  }).catchError((e) {
    print("inside catchError");
    print(e.error);
  });
}
//  Output:
//    inside then
//    [waited 1000 ms, waited 1000 ms, waited 2000 ms]
{% endhighlight %}

But, even a single error from `pause()` makes `Future.wait()` return an error:

{% highlight dart %}
void main() {
  // pause(1200) throws
  Future.wait([pause(1000), pause(1200), pause(2000)]).then((value) {
    print("inside then");
    print(value);
  }).catchError((e) {
    print("inside catchError");
    print(e.error);
  });
}

//  Output:
//    inside catchError
//    Illegal argument(s): only whole seconds, please
{% endhighlight %}

Using `Future.main()` is something of an all-or-nothing proposition; there 
is no room for find-grained error handling. 
