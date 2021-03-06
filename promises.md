# Promises

This is a discussion of promises. It covers how they can be used to represent asyncronous values and error handling. There is some debate over whether it should be included in the language, and if so when.  There is also considerable discussion of terminology. `.then` seems to be mostly agreed on in preference to `.when` but there is still considerable debate about the overloaded terms `promise`, `future` and `deferred`.

## David Bruant

[_Tue Nov 6 10:47:17 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026188.html)

In a [post](http://lists.w3.org/Archives/Public/public-script-coord/2012OctDec/0122.html) to public-script-coord yesterday, Alex Russel wrote the following:

> [Web]IDL is handy. More to the point, it's the language of the specs we have now, and the default mode for writing new ones is "copy/paste some IDL from another spec that looks close to what I need and then hack away until it's close". This M.O. is exacerbated by the reality that most of the folks writing these specs are C++ hackers, not JS developers. For many, WebIDL becomes a safety blanket that keeps them from having to ever think about the operational JS semantics or be confronted with the mismatches.

I wasn't aware of this and then read through about a [dozen WebAPIs](https://wiki.mozilla.org/WebAPI#APIs) between yesterday and today and... discovered it's the case. In my opinion, one of the most absurd example is the DOMRequest thing which looks like:

    {
      readonly attribute DOMString readyState; // "processing" or "done"
      readonly attribute DOMError? error;
      attribute EventListener      onsuccess;
      attribute EventListener      onerror;
      attribute readonly any?      result;
    };

Read it carefully and you'll realize this is actually a promise... but it has this absurd thing that it has to have both an error and result field while only one is actually field at any given point.Also, these APIs and JavaScript as it they are won't support promise chainability and the excellent error forwarding that comes with it off-the-shelf. Also, the lack of a built-in Q.all really doesn't promote good code when it comes to event synchronization. Oh yes, of course, you can always build a promise library on top of the current APIs, blablabla... and waste battery with [these libraries](http://assets.en.oreilly.com/1/event/79/Who%20Killed%20My%20Battery_%20Analyzing%20Mobile%20Browser%20Energy%20Consumption%20Presentation.pdf).

I'm coming with the following plan:

1. get promises in ECMAScript
2. get WebIDL to support ECMAScript promises
3. get browser APIs to use WebIDL promises

About the first step, there is a [strawman](http://wiki.ecmascript.org/doku.php?id=strawman:concurrency) that contains promises and it requires to define the event loop, so that's probably too much for ES6. Yet, it doesn't prevent to agree on a promise API that will be adopted in ES7. Besides the strawman, promises have run a long way from [CommonJS](http://wiki.commonjs.org/wiki/Promises) to [jQuery](http://api.jquery.com/category/deferred-object/) to [Q](https://github.com/kriskowal/q) to Irakli's [excellent post](http://jeditoolkit.com/2012/04/26/code-logic-not-mechanics.html) to Domenic's [recent rant](https://gist.github.com/3889970) and I've missed a lot of other examples probably. The JS community is ready for promises. The idea has been used a lot. Different libraries have different APIs and I have no preference. The only things I really care about is chaining (with error forwarding) and a promise-joining function &agrave; la `Q.all`. I'll let people who care about naming fight.

I'm sure TC39 can come to an agreement *before* ES7 standardization, 
agreement that can be used by WebIDL and browser APIs (why not 
implemented long before ES7 work has even started).
If you're a JS dev and care about promises, please show some support to 
this proposal :-)

## Domenic Denicola

[_Tue Nov 6 10:59:36 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026189.html)

I guess now is a good a time as any to pre-announce Promises/A+:

https://github.com/promises-aplus/promises-spec

It’s an attempt to improve significantly on the minimal-but-perhaps-too-minimal Promises/A of CommonJS, making the language more rigorous and speccing several important things Promises/A missed. Here’s a short summary of the differences:

https://github.com/promises-aplus/promises-spec/issues/17

The two most important ones, in my opinion, are:

 * Cover the case of handlers returning a promise (chaining)
 * Require asynchronous resolution

Promises/A+ is a collaborative effort led by Brian Cavalier (when.js) with help from myself and Kris Kowal (Q), as well as Yehuda Katz (rsvp.js, TC39) and others with whom I am less personally familiar with but have also been very helpful.

---

I say "pre-announce" because there are a number of bookkeeping issues we want to take care of before saying it’s truly done:

https://github.com/promises-aplus/promises-spec/issues

But everything important is already in the repo. We also have a (again, preliminary) conformance test suite at

https://github.com/promises-aplus/promises-tests

---

I hope this is helpful to TC39 or others considering promise standardization. My ideal vision is that the community experiments with promises + generators (a la taskjs) in the ES6 timeframe, then in ES7 we standardize on something like the concurrency strawman or a C#-like async/await pattern based on promises.

We welcome feedback from TC39 and others, preferably as GitHub issues in the repo (and thus we can avoid derailing this thread). We’d especially love the eyes of spec-experienced folks such as those that frequent this list.

## Axel Rauschmayer

[_Tue Nov 6 11:07:40 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026190.html)

That’s at a weird intersection between HTML5 and ECMAScript, but it would be great to have!

I recently played a little more with IndexedDB and found it complicated to use:

1. In contrast to the Web SQL database API and the MongoDB API when it comes to functionality
2. In contrast to Node.js callbacks (where input and output are clearly separated) and promises when it comes to invocation mechanics

It might just be me "not getting it", but there must be ways to make things more elegant (on both accounts).

## David Bruant

[_Tue Nov 6 11:15:38 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026191.html)

> That’s at a weird intersection between HTML5 and ECMAScript, (...)

I think it's more historical than anything. The event loop, `setTimeout`/`setInterval` (and promises) belong to the language (ECMAScript), not to a library (HTML5) in my opinion.  ECMAScript 1-5 were concurrency-neutral. A pure ES5 programs has a start and an end and that's it. No concurrency whatsoever.  ES6 is going in that direction too.  ES7 opens a breach to event loop concurrency with async observers (object.observe)

## Rick Waldron

[_Tue Nov 6 11:35:00 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026192.html)

Based on a read through of [promises-aplus/promises-spec](https://github.com/promises-aplus/promises-spec), these things initially come to mind, please regard as a loose collection of varying thoughts that may or may not be completely relevant:

1. The definition of a "promise" is really just a plain object or function with an expando property, I would think that a language level addition would require its own standard built-in object: Promise, which when invoked as a constructor initializes a new promise object which has a `then` method... Domenic has it covered from there.
2. The notes describe some excellent practical implementation points, but none of them are actually part of the ECMAScript standard, eg. `setTimeout`, `process.nextTick`. Should these be specified or left unspecified?  Object.observe describes delivery as "Schedule change events to be delivered asynchronously 'at the end of the turn'", which is not very
specific.
3. Does this belong in the language or would it make more sense to exist as a "standard module"?

## David Bruant

[_Tue Nov 6 11:43:27 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026193.html)

> Based on a read through of [promises-aplus/promises-spec](https://github.com/promises-aplus/promises-spec), these things initially come to mind, please regard as a loose collection of varying thoughts that may or may not be completely relevant:
>
> 1) The definition of a "promise" is really just a plain object or function with an expando property, I would think that a language level addition would require its own standard built-in object: Promise, which when invoked as a constructor initializes a new promise object which has a `then` method... Domenic has it covered from there.

I fully agree.

> 2) The notes describe some excellent practical implementation points, but none of them are actually part of the ECMAScript standard, eg. `setTimeout`, `process.nextTick`. Should these be specified or left unspecified?  Object.observe describes delivery as "Schedule change events to be delivered asynchronously 'at the end of the turn'", which is not very
specific.

As I suggested, Object.observe opens the breach and I think it means the 
event loop (including the notion of "turn") will have to be fully 
specified within ECMAScript.

> 3) Does this belong in the language or would it make more sense to exist as a "standard module"?

Are you referring to the event loop or promises?
event loop : the language
promises : arguably standard module

## Domenic Denicola

[_Tue Nov 6 15:05:20 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026201.html)

As an interesting aside, I just wanted to highlight the section of Promises/A+ that Rick was referring to, because I think it uses a rather clever trick to avoid discussing the event loop while still requiring the behavior we want:

"onFulfilled and onRejected must not be called before then returns [1]."

This trick is borrowed from Kris Kowal's UncommonJS promises specification.

## Mikeal Rogers

[_Tue Nov 6 12:33:51 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026196.html)

> The definition of a "promise" is really just a plain object or function with an expando property, I would think that a language level addition would require its own standard built-in object: Promise, which when invoked as a constructor initializes a new promise object which has a `then` method... Domenic has it covered from there.

also, node.js won't adopt either a promise "API" or a promise syntax for it's core API. if it lands in the language then nothing is stopping people from using it but the ecosystem is highly unlikely to adopt it either since it's not uniform across node.

we've had great success using function (err, result) {} across core and across the ecosystem, it's been widely successful as you can see by the number of modules created, the breath of functionality those modules provide, and their seamless compatibility with each other.

## Domenic Denicola

[_Tue Nov 6 12:43:14 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026197.html)

> also, node.js won't adopt either a promise "API" or a promise syntax for it's core API. if it lands in the language then nothing is stopping people from using it but the ecosystem is highly unlikely to adopt it either since it's not uniform across node.

Um:

https://twitter.com/izs/status/257634118320410624

> Having node cb-taking methods return a promise is not out of the question. 

## Mikeal Rogers

[_Tue Nov 6 13:13:28 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026198.html)

Having two compatibility modes for IO would not be beneficial to the ecosystem, we can argue that on and on and i'll pull ry out of "retirement" before i concede to it going to in to core :)

## David Herman

[_Tue Nov 6 11:55:29 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026194.html)

I agree that promises should be standardized in Ecma-262. There are a number of subtleties that'll need to be hashed out:

- the tension between the flexible structural ("duck") type and the desire to have the semantics do different things based on dynamically testing "whether" something is a promise:
- the common practice (sadly not at all spelled out in Promises/A) of treating a resolved value that is a promise as continuing to defer resolution
- the `when` operation that either calls its callback synchronously or asynchronously depending on whether it's a promise
- whether it's a bad idea (*cough* it is *cough*) for `when` to do that
- whether promises ever call their callbacks synchronously
- every single corner case of when an error is thrown and where it is thrown
- the interaction with the event-loop semantics, which needs to be specified in ES6 but isn't yet written
- whether the `then` property name should be a symbol or a string
- a way to create promises that don't expose their internal "resolve me" methods, etc. so they can be delivered to untrusted clients, e.g.:
```javascript
var [internalView, externalView] = Promise.makePair();
"resolve" in internalView // true
"resolve" in externalView // false
```

I agree this is worth doing, though definitely post-ES6. Personally I would like to work on it but for me anyway it has to take a back seat to modules and binary data.

## Erik Arvidsson
[_Tue Nov 6 12:18:50 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026195.html)

> I agree that promises should be standardized in Ecma-262.

Yes.

> A way to create promises that don't expose their internal "resolve me" methods, etc. so they can be delivered to untrusted clients, e.g.:
>
>```javascript
>var [internalView, externalView] = Promise.makePair();
>"resolve" in internalView // true
>"resolve" in externalView // false
>```

This is very important. You don't want the consumer, of XHR for example, to resolve the promise. It is not only about trusted clients, it is also about keeping the API clean.

> I agree this is worth doing, though definitely post-ES6. Personally I would like to work on it but for me anyway it has to take a back seat to modules and binary data.

I would say high priority for ES7. Champions wanted. Alex, I'm calling on you!

## Tom Van Cutsem

[_Tue Nov 6 23:11:22 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026209.html)

I'm happy to assist with a strawman/spec for promises. I too would like to see them standardized.

## Claus Reinke

[_Tue Nov 6 15:42:06 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026203.html)

> I agree that promises should be standardized in Ecma-262. 

Agreed. That would also offer the possibility to support promises in syntax. In ES7 (latest), I would like to see something roughly like

    { ...;
      let x <- promise;
      ...;
    }

(read as `let x from promise`), desugaring into

    { ...;
      promise.then( (x) => { ...; };
    }

Essentially, this is an even shallower continuation than generators (only the remaining statements in the current statement list), but it is already sufficient to avoid callback nesting issues in async code. It also suffices to implement generators, but for syntax. 

By relying on nothing but a `.then` method, this isn't tied to promises in the narrow sense, but provides a reusable building block for other control abstractions (to begin with, abstracting away the default error handling in some APIs).

## Axel Rauschmayer

[_Tue Nov 6 16:07:15 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026205.html)

Why not task.js, instead?

Different issue: do we already have a solution for a missing error handler causing silent failures? task.js should cover this, too (right?)

## Domenic Denicola

[_Tue Nov 6 16:13:41 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026207.html)

> Different issue: do we already have a solution for a missing error handler causing silent failures? task.js should cover this, too (right?)

Yes, since task.js essentially adds fulfillment and rejection handlers everywhere automatically, it's impossible to be left with an unhandled rejection.

(This assumes you use `yield` on all your promises, but then again, if you don't, you are essentially signaling that you don't care about the result.)

## Claus Reinke

[_Wed Nov 7 01:18:34 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026210.html)

> Why not task.js, instead?

1. Implementation complexity

    let-from: trivial syntax transformation

    task.js+generators: non-trivial syntax transformation or
        -more likely- non-trivial runtime system manipulation

2. Usage complexity

    let-from: just another statement in the block,
        merely syntax for conventional promise/callback-style

    task.js+generators: spawn+function*+yield,
        complex rts + complex library

3. Usage granularity

    let-from: statement level

    task.js+generators: function level

Note that the trade-offs are not entirely against task.js: because JS
has so much non-overridable surface syntax, it is useful to have the
pre-packaged functionality of generators built-in, and it is useful to
reuse that functionality for async and co-routine code, via task.js.
As long as you use it to wrap non-trivial segments of code.

But when you don't want to deal with code that uses while/for/if/..,
or if you want to build up your code patterns from smaller
components, then let-from has advantages. For let-from to fully
replace task.js and generators, JS would need library-based
control structures or overridable control-structure syntax (eg,
Haskell monads or F# computation expressions).

## Andreas Rossberg

[_Wed Nov 7 03:11:08 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026211.html)

> a way to create promises that don't expose their internal "resolve me" methods, etc. so they can be delivered to untrusted clients, e.g.:
>
>```javascript
>var [internalView, externalView] = Promise.makePair();
>"resolve" in internalView // true
>"resolve" in externalView // false
>```

Indeed. I think this is an issue where many promise/future libraries
are confused/broken. FWIW, when creating a concurrent language called
Alice ML some 15 years ago we thought about this quite extensively,
and ended up introducing the following separation of concepts:

* Futures are handles for (potentially) unresolved/asynchronous
values, on which you can wait and block -- but you cannot directly
resolve them.

* Promises are explicit resolvers for a future. More specifically,
creating a promise creates an associated future, which you can safely
pass to other parties. Only the promise itself provides the fulfill
method (and related functionality) that enables resolving that future.

In other words, futures provide synchronisation, while promises
provide resolution.

Incidentally, that's also exactly the model and naming that C++11 picked.

## Kevin Smith

[_Wed Nov 7 06:58:20 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026213.html)

> In other words, futures provide synchronisation, while promises
> provide resolution.
>

This is exactly the API that Q (and it's derivatives) use, although the nomenclature is different.  In Q, the "future" is called a promise, and the "promise" is what you get from calling `defer()`:

```javascript
let { resolve, reject, promise } = Q.defer();
```

I think the nomenclature you've provided is superior.  Using those names we'd have an API that looks something more like this:

```javascript
let promise = new Promise();
let future = promise.future;

// Futures have a then method, ala Promises/A+
future.then(val => { ... });

// Promises are resolved using methods on the promise object:
promise.resolve(val);
promise.reject(val);
```

The aesthetic issue I have with Promises/A+ is that the error handling interface is ugly:

```javascript
future.then(val => {
  ...
}, err => {
  ...
});
```

In all of our APIs we try to avoid placing callbacks in non-terminating argument positions, which this violates.

The only hard part that isn't really addressed by currently library implementations is error handling.  I feel pretty strongly that rejections (and by extension, errors thrown from `then` callbacks), should ALWAYS be observable.  In other words, if a rejection is not observable via error listeners at the time when error listeners should be called, then an unhandled error should be thrown by the runtime.

In my usage of promises I have never wanted anything other than this behavior.

## Domenic Denicola

[_Wed Nov 7 07:50:43 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026215.html)

> The only hard part that isn't really addressed by currently library implementations is error handling.  I feel pretty strongly that rejections (and by extension, errors thrown from `then` callbacks), should ALWAYS be observable.  In other words, if a rejection is not observable via error listeners at the time when error listeners should be called, then an unhandled error should be thrown by the runtime.

> In my usage of promises I have never wanted anything other than this behavior.

The problem with this is that it disallows treating promises as first-class objects that can be passed around, unobserved, only to be observed at a later date. As a trivial example, consider:

```javascript
let promise = Q.reject(); // a rejected promise

setTimeout(=> {
    promise.then(null, (err) => {
        console.err("Got an error!", err);
    });
}, 100);
```

In this example there is nobody observing the promise at the time it is rejected, or even a tick after rejection. So should the rejection be thrown by the runtime? You would suggest yes. But then the error handling code inside the `setTimeout` will never be called.

---

OK, so that's a trivial example. What about a less trivial example? Well, consider using promises as remote objects. A rejected promise could be passed across the wire in various ways, all of which take much longer than a single tick. Or consider just normal control flow that uses promises as first-class mechanisms of state. For example the upthread-mentioned promises in place of loading events: many libraries will only end up listening to loading events far after they are completed, since the loading promise is a first-class observable property of the object being loaded (page, image, database, etc.).

In short, it creates a serious refactoring hazard. If you accept a promise, you can no longer introduce asynchronicity into your functions that handle it (ironic!) due to the risk of its errors escaping you:

```javascript
function processData(promiseForDatabase) {
    // All good:
    return promiseForDatabase.then(=> ..., => ...)
}

function processData(promiseForDatabase) {
    // No good!! You lost the state of `promiseForDatabase`. Its errors have escaped,
    // possibly crashing your app if you are e.g. in a Node.js or Windows 8 Metro environment.
    return getDataForPreprocessing().then(=> {
        return promiseForDatabase.then(=> ..., => ...);
    });
}
```

---

As mentioned upthread, this is solved by task.js, but this is by far the thorniest issue faced by promise implementations today. In Q and WinJS, the solution is to always "cap" your promise chains with `.done()`. So all promise code should either be returning the promise, or capping with `.done()`. Other mechanisms we are considering are mostly about enabling greater visibility into any currently-unhandled rejections. For example, some type of Q.onunhandled/Q.onhandled pair, or integration into some simple browser-console extensions that would create a pane where you could view such rejected promises, or a mode that sets a maximum timeout before we consider an unhandled rejection erroneous and throw it (for development purposes), or somehow showing the errors on "exit" (page unload, process exit in Node, ...).

## Tom Van Cutsem

[_Wed Nov 7 08:57:54 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026217.html)

>
> In other words, futures provide synchronisation, while promises provide resolution.
>
> This is exactly the API that Q (and it's derivatives) use, although the nomenclature is different.  In Q, the "future" is called a promise, and the "promise" is what you get from calling `defer()`:
>
>```javascript
>let { resolve, reject, promise } = Q.defer();
>```
>
> I think the nomenclature you've provided is superior.

While we're talking nomenclature: the terms "promise" and "future" also appear, with roughly the semantics described by Andreas in [Scala's API](http://docs.scala-lang.org/sips/pending/futures-promises.html) and [Clojure's API](http://www.michaelharrison.ws/weblog/?p=239) (both very recent APIs). I know MarkM dislikes the
use of these terms to distinguish synchronization from resolution, as he
has long been using those same terms to distinguish traditional "futures",
which provide a .get() method blocking the calling thread and returning the
future's value when ready (as in e.g. Java), from "promises", which only
provide a non-blocking "when" or "then" method requiring a callback, never
blocking the event loop thread (as in all the JavaScript promise APIs).

To my mind, the term "future" is still very closely tied to blocking synchronization. YMMV.

There's a [helpful Wikipedia article](http://en.wikipedia.org/wiki/Futures_and_promises) that compares and contrasts some uses
of the terms.

## Andreas Rossberg

[_Wed Nov 7 11:12:46 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026232.html)

I see. Interesting, I wasn't aware of Mark's reservations :). Mark, is that just about the terminology, or also conceptually?

(Please correct me if I'm wrong, though, IIRC, the original Friedman &
Wise article introduced the term "promise" for something that's rather
a future according to that distinction.)

## Mark S. Miller

[_Wed Nov 7 13:19:03 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026233.html)

It is just terminology. Prior to E, the closest similar system was [Liskov & Shrira's](http://dl.acm.org/citation.cfm?id=54016), which called them "promises". All the non-blocking promise systems I am aware of, with the exception of Tom's AmbientTalk, have called them promises or deferreds. AFAIK, all are derived from E's promises or Liskov & Shrira's promises. I think we should respect this history; but history itself is not a strong argument.

The reason I like the "promise" terminology is that it naturally accounts for the three main states of a promise: unresolved, fulfilled, and broken. A major feature of many "promise" systems (including IIRC Liskov and Shrira's) that I do not recall being implemented by "future" systems (with the exception of Tom's) is this broken state, as well as the broken promise contagion rules which go with it.

## Andreas Rossberg

[_Mon Nov 12 08:31:39 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026283.html)

> The reason I like the "promise" terminology is that it naturally accounts
> for the three main states of a promise: unresolved, fulfilled, and broken.

I see. Of course, though, in the holder/resolver approach, those states jointly apply to both objects. My reasoning is that in that approach, then, the name "promise" is more suitable for the resolver object, because that's what has the "fulfill" and "fail" methods. The other only has "then"/"when" and friends, which is why a temporal name like "future" is kind of intuitive.

But I understand your argument about history and terminology. I can get rather worked up about abuses of pre-established terminology. I don't dare mention my pet peeves on this list. :)


> A major feature of many "promise" systems (including IIRC Liskov and Shrira's)
> that I do not recall being implemented by "future" systems (with the
> exception of Tom's) is this broken state, as well as the broken promise
> contagion rules which go with it.

Maybe I misunderstand, but MultiLisp already had a notion of failed future, I think, even if it wasn't really discussed in their paper. It is kind of inevitable once you combine the future (or promise) idea with exceptions. Consequently, it also is part of the future semantics of at least Oz, Alice ML, Scala, and C++.

## Mariusz Nowak

[_Thu Nov 8 03:45:48 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026240.html)

Kevin Smith wrote:
> 
> The only hard part that isn't really addressed by currently library
> implementations is error handling.  I feel pretty strongly that rejections
> (and by extension, errors thrown from `then` callbacks), should ALWAYS be
> observable.  In other words, if a rejection is not observable via error
> listeners at the time when error listeners should be called, then an
> unhandled error should be thrown by the runtime.
> 
> In my usage of promises I have never wanted anything other than this
> behavior.
> 
> 

I think source of a problem is that we center usage of promises just around `then` function, when `then` is about two things:

1. Provides us with resolved value
2. Extends promise chain with another promise.

What's important, in many use cases we're not after point 2, we don't need extended promise, and it's promise extension fact that, makes our errors silent when we don't expect them to be.

It's difficult to naturally expose errors when the only way to add observers is `then`. Technically to do it we need to always write error handler as below:

```javascript
promise.then(function () {
  // process the value;
}).then(null, function (err) {
  // Need to get out of promise chain with setImmediate (or nextTick if in Node.js)
  setImmediate(function (function () {
    throw err; // Finally error will be thrown naturally;
  });
});
```

This one of the reasons for which some developers preferred to stay away from promises, and I totally I agree with them.

Q implementors spotted that issue, and provided `done` (initially named as `end`) function. Which helps to work with that issue:

```javascript
promise.then(function () {
 // process the value
}).done(); // Sugar for above
```

Still in Q (as far I as know) we're not able to get to resolved value without extending the promise chain and that's not great.

Final conclusion is that there needs to be a way to observe resolved value on promise without extending the chain as `then` does.

And yes there is library that does that. In [deferred implementation](https://github.com/medikoo/deferred) I've solved it by providing two other functions that have same signature as 'then' but *don't extend* promise chain:

```javascript
promise.end(onFulfilled, onRejected); // returns undefined
```

If `onRejected` is not provided then failed promise will throw, additionally any errors that may occur in provided callbacks are thrown natural way (they're not caught by promise implementation)

```javascript
promise.aside(onFulfilled, onRejected); // returns self promise
```

This actually works similar to functions found in jQuery's Deferred. It's useful when we want to return same promise, but on a side, work with resolved value. If onRejected is not provided nothing happens (as we return promise for further processing), but any errors that occur in callbacks are thrown natural way  (they're not caught by promise implementation)

With such design your function of choice in first place should always be `end`, in that case there is no problem with silent errors. `then` should be used *only* if you have a reason to extend the chain, and pass result elsewhere.

## Domenic Denicola

[_Thu Nov 8 03:58:06 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026241.html)

On Nov 8, 2012, at 6:45, "Mariusz Nowak" <medikoo+mozilla.org at medikoo.com> wrote:

> Q implementors spotted that issue, and provided `done` (initially named as
> `end`) function. Which helps to work with that issue:
> 
>```javascript
>promise.then(function () {
>  // process the value
>}).done(); // Sugar for above
>```
> 
> Still in Q (as far I as know) we're not able to get to resolved value
> without extending the promise chain and that's not great.

If I understand correctly, `promise.done(onFulfilled, onRejected)` does what you want in Q and in WinJS. See

http://msdn.microsoft.com/en-us/library/windows/apps/hh700337.aspx

for a nice explanation from the WinJS folks.

## Mariusz Nowak

[_Thu Nov 8 04:10:45 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026242.html)

When I scanned you're API not so long ago, it was as I described above. It
looks you've changed the behavior with one of the recent v0.8 rollouts and
that's definitely a good decision :)

## Alex Russell

[_Wed Nov 7 04:53:50 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026212.html)

Sorry for ignoring the rest of this thread in my first reply, but I'll try
to cover as much ground as I can here. Response inline:



On Tue, Nov 6, 2012 at 6:47 PM, David Bruant wrote:

> In a [post](http://lists.w3.org/Archives/Public/public-script-coord/2012OctDec/0122.html) to public-script-coord yesterday, Alex Russel wrote the following:
>
>> [Web]IDL is handy. More to the point, it's the language of the specs we have now, and the default mode for writing new ones is "copy/paste some IDL from another spec that looks close to what I need and then hack away until it's close". This M.O. is exacerbated by the reality that most of the folks writing these specs are C++ hackers, not JS developers. For many, WebIDL becomes a safety blanket that keeps them from having to ever think about the operational JS semantics or be confronted with the mismatches.
> 
> I wasn't aware of this and then read through about a [dozen WebAPIs](https://wiki.mozilla.org/WebAPI#APIs) between yesterday and today and... discovered it's the case. In my opinion, one of the most absurd example is the DOMRequest thing which looks like:
>```
>{
>  readonly attribute DOMString readyState; // "processing" or "done"
>  readonly attribute DOMError? error;
>  attribute EventListener      onsuccess;
>  attribute EventListener      onerror;
>  attribute readonly any?      result;
>};
>```

>Read it carefully and you'll realize this is actually a promise... but it has this absurd thing that it has to have both an error and result field while only one is actually field at any given point.

There are absurdities like this all over DOM for want of promises. A short
list must include:


   - XHR's "readystate" system
   - geolocation APIs
   - window.onload, DOMContentLoaded, etc.
   - CSS OM measurement APIs.

I'm sure I'm missing some. I'm also aware of new APIs that could be re-cast
in terms of Promises/Futures/whatevs to good effect.


> Also, these APIs and JavaScript as it they are won't support promise chainability and the excellent error forwarding that comes with it off-the-shelf. Also, the lack of a built-in Q.all really doesn't promote good code when it comes to event synchronization. Oh yes, of course, you can always build a promise library on top of the current APIs, blablabla... and waste battery with [these libraries](http://assets.en.oreilly.com/1/event/79/Who%20Killed%20My%20Battery_%20Analyzing%20Mobile%20Browser%20Energy%20Consumption%20Presentation.pdf).
>
> I'm coming with the following plan:
> 
> 1. get promises in ECMAScript
> 2. get WebIDL to support ECMAScript promises
> 3. get browser APIs to use WebIDL promises

From a "doability" perspective, I think this is backwards. Browser vendors
are more likely to add ad-hoc APIs, and the large problem of DOM design is
that Promises aren't in the WebIDL "toolchest". Further, this group is even
more gunshy than many W3C WGs to make progress in important areas for fear
of backwards compatibility burdens. It's also harder to iterate ES since it
moves so slowly (in relative spec-org terms; all spec processes move slowly
in human terms).

My plan, as a result, inverts yours:

   1. Get "DOMPromises" done in order to fix some busted proposed API. My
   current hope is WebCrypto who can both avoid a design catastrophe and
   introduce a new, widely implemented API on a relatively short timeframe
   2. Once we have DOMPromises implemented, we advocate broader
   use throughout DOM APIs.
   3. Introduce ES7 Promises as a compatible subset of DOMPromises

In an ideal world we'd go your route (as we would have with
Object.observe() vs. Mutation Observers), but TC39 isn't known for adding
API quickly, no matter how popular or well-argued the case.

> About the first step, there is a [strawman](http://wiki.ecmascript.org/doku.php?id=strawman:concurrency) that contains promises and it requires to define the event loop, so that's probably too much for ES6. Yet, it doesn't prevent to agree on a promise API that will be adopted in ES7. Besides the strawman, promises have run a long way from [CommonJS](http://wiki.commonjs.org/wiki/Promises) to [jQuery](http://api.jquery.com/category/deferred-object/) to [Q](https://github.com/kriskowal/q) to Irakli's [excellent post](http://jeditoolkit.com/2012/04/26/code-logic-not-mechanics.html) to Domenic's [recent rant](https://gist.github.com/3889970) and I've missed a lot of other examples probably. The JS community is ready for promises. The idea has been used a lot. Different libraries have different APIs and I have no preference. The only things I really care about is chaining (with error forwarding) and a promise-joining function &agrave; la `Q.all`. I'll let people who care about naming fight.
>
> I'm sure TC39 can come to an agreement *before* ES7 standardization, agreement that can be used by WebIDL and browser APIs (why not implemented long before ES7 work has even started). If you're a JS dev and care about promises, please show some support to this proposal :-)

I support getting Promises done where they'll make an impact and right now
DOM is the squeeky wheel. Yes, we need them in JS, but getting the eyes
around this particular table opened to that is a fight I don't want right
now (as the rest of this thread is painful, painful proof).

## Brendan Eich

[_Wed Nov 7 07:48:57 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026214.html)

Alex Russell wrote:
> My plan, as a result, inverts yours:
>
>  1. Get "DOMPromises" done in order to fix some busted proposed API.
>     My current hope is WebCrypto who can both avoid a design
>     catastrophe and introduce a new, widely implemented API on a
>     relatively short timeframe
>  2. Once we have DOMPromises implemented, we advocate broader
>     use throughout DOM APIs.
>  3. Introduce ES7 Promises as a compatible subset of DOMPromises
>

I agree with this approach provided TC39 (not just you) keeps tracking and commenting. public-script-coord is not overused...

> In an ideal world we'd go your route (as we would have with 
> Object.observe() vs. Mutation Observers), but TC39 isn't known for 
> adding API quickly, no matter how popular or well-argued the case.

That's truthy for some good reasons: programming language over library 
expertise, less domain expertise at the core.

It's also kind of false, in that ES5 added a lot of API, but we ended up 
with regrets. You (broadly speaking; DOM/w3/web/sysAPI people) will too. 
Going fast inevitably means adding APIs with flaws that will be hard to 
fix if the APIs are adopted on the web. HTML5 has more than a few such 
APIs, even discounting IDL effects.

Chrome as well as other browsers will not want to break compatibility, 
so we'll be stuck with flawed APIs. The web standards process must keep 
on composting ;-).

Bottom line: it's a good thing for domain experts who also have good API 
chops to lead the way on DOMPromises. But do keep es-discuss (David, 
Domenic, et al.) and TC39 (tomvc, especially) in the loop. Probably we 
will meet in the middle, in ES7.

## David Herman

[_Wed Nov 7 10:07:15 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026230.html)

On Nov 7, 2012, at 7:48 AM, Brendan Eich <brendan at mozilla.com> wrote:

>> 1. Get "DOMPromises" done in order to fix some busted proposed API.
>>    My current hope is WebCrypto who can both avoid a design
>>    catastrophe and introduce a new, widely implemented API on a
>>    relatively short timeframe
>> 2. Once we have DOMPromises implemented, we advocate broader
>>    use throughout DOM APIs.
>> 3. Introduce ES7 Promises as a compatible subset of DOMPromises
>> 
> 
> I agree with this approach provided TC39 (not just you) keeps tracking and commenting. public-script-coord is not overused...

+1

> Bottom line: it's a good thing for domain experts who also have good API chops to lead the way on DOMPromises. But do keep es-discuss (David, Domenic, et al.) and TC39 (tomvc, especially) in the loop. Probably we will meet in the middle, in ES7.

I support this plan. Please keep me in the loop.

## David Bruant

[_Fri Nov 9 04:33:50 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026248.html)

Hi,

In this message, I'll be sharing some experience I've had with the Q library. I have worked with it for about 7-8 months in a medium/big Node.js application (closed source, so I can't link, sorry).  I'll be covering only the parts that I have used during this experience. It's related to my own writing style and I don't mean that the rest is 
useless and should be thrown away, but the subset I'll be covering here has proven to be sufficient to my needs for several months.

I would be interested if others could share their experience if they had a different way of using promises.

### the Q API

#### A Q Deferred is a {promise, reject, resolve} object.

Only the deferred holder can resolve the promise (and not the promise holder), 
addressing the security issue that's been raised on the list.
You create a Deferred instance by calling `Q.defer()`

Typically, when trying to promise-ify async APIs, you end up doing 
something like (and that's probably also how things are done internally):

```javascript
function query(myQuery){
  var def = Q.defer();

  db.query(myQuery, function(err, data){
    if(err)
      def.reject(err)
    else
      def.resolve(data)
  })

  return def.promise;
}
```

#### A Q Promise is a {then, fail} object.

Given the previous code, it's possible to do:

```javascript
var someData1P = query(q1);
someData1P.then(function(data){
  // do something with data
});
someData1P.fail(function(error){
  // handle the error
});
```

Both then and fail return another promise for the result of the callback

```javascript
var someData1P = query(q1);
var processedDataP = someData1P.then(function first(data){
    return process(data); // if what's returned here is a promise P, then
         // processedDataP will be a promise for the resolution value of P, not
         // a promise for a promise of the resolution value of P
         // Said another way, in a .then callback, the argument is always a non-promise value
  });

var processedAgain = processedDataP.then(function second(processedData){
    // play with processed data
  });
```

Of course, this second call returns a promise too, which I'm free to ignore.
If a database error occurred, neither the first nor the second callback 
will be called. Thanks to chaining, one interesting part is that I can 
hook a .fail only to th last promise in the chain to process the error. 
Following previous code:

```javascript
processedAgain.fail(function(err){
  // handle error, even if it's an error as "old" database error
});
```

This is very close to how throw & try/catch work where you don't always 
need to handle the error as it happens, but you can catch it if no one 
else before you did. It's possible to forward an error by re-throwing in 
inside the callback. The above code could look like:

```javascript
var someData1P = query(q1);
var processedDataP = someData1P.then(function first(data){
  return process(data);
}).fail(function(err){
  // process err at this level and forward it.
  throw err;
})

var processedAgain = processedDataP.then(function second(processedData){
  // play with processed data
}).fail(function(err){
  // process the error
});
```

In this case, "intermediate" promises are generated (before the `.fail`). 
I don't think this is too high of an overhead for the excellent 
readability it provides.

The fact that the `.then` of the next promise is called when the previous 
normally returned and the `.fail` when the previous threw makes me feel 
like promise chains have sort of two parallel channels to which one 
decides to branch to by returning or throwing.

I agree with what Kevin Smith said about the Promises/A+ aesthetic 
issue. The functions passed to `.then` and `.fail` are often function 
expressions (for me, the only exception was some final `.fail` callbacks 
for which an error handling function had been prepared in advance and 
was reused). I feel that when you have two function expressions 
separated only with a comma (to separate the onsuccess and onerror 
arguments), it's less easily readable than when you have your function 
expression prefixed with `.then(` or `.fail(`. That's mostly writing 
style and aestetics so I won't be fighting to death to have both 
separated, but it feels like noticeable enough to be noted.


#### Q.all

My favorite feature is the `Q.all` function. `Q.all` accepts an array of 
promises and returns a promise which will be fulfilled when all promises 
are. The resolution values are the different promises resolution values:

```javascript
var someData1P = query(q1);
var someData2P = query(q2);
var someData3P = fetch(url); // HTTP GET returning a promise

Q.all([someData1P, someData2P, someData3P])
  .spread(function(someData1, someData2, someData3){
    // do something when all data are back
  })
```

I used this extensively and it's been extremely helpful. Personally, to 
synchronize different async operations, I've never read code more 
elegant than what `Q.all` offers. I'm interested in hearing what other's 
experience is on that point.
Arguably, `Q.all` could take several arguments instead of accepting only 
an array (that's one thing I'd change). Maybe there is a good reason to 
enforce an array, but I don't know it.

I think the `.fail` is called if any promise is broken you get only the 
first error as a result of `Q.all`, but you can always inspect each 
promise individually if you care about all errors. It never happened to 
me. Usually, when I did `Q.all`, I cared if all were successful or if one 
was broken, but several broken was not a use case I cared about. So, 
`Q.all` covered the 80% case (well, actually 100%) for me.

### Debugging

It's been said in other messages, one part where Q promises fell short 
was debugging. With thrown errors, if you uncatch one, your 
devtools/console will tell you. To my experience, with the Q library, if 
you forget a `.fail`, an error may end up being forgotten which is bad 
news in development mode (and I've wasted a lot of times not knowing an 
error had happened and chasing this errors after understanding that's 
why "nothing" was happening). I'm hopeful built-in promises will be able 
to compensate.

Basically, when a promise wasn't used to generate a new promise (to 
forward the error to), and ends up broken, you know you're facing an 
unhandled broken promise. The complicated (undecidable-style 
complicated) question is "how can one knows whether a promise will not 
be used anymore?"
For sure, a GC'ed promise that hasn't been used to generate a new 
promise won't generate a new one, so if it's broken, it's clearly an 
unhandled broken.
I've seen the promise.end and promise.aside in Mariusz Nowak post and 
both are very interesting. Specifically, `.end` is a way for developers to 
say "this promise chain is over" which devtools can easily interpret as 
"this promise won't forward the error any longer, I can report it as 
uncaught (if uncaught)". For built-in promises, there is no need to 
throw on failed promises lacking an onrejected I think (as it may be for 
a library implementing `.end`).

For un-GC'ed promises with no `.end`, I don't know what can be done. Maybe 
keep a record of all still-unhandled broken promises and have this 
accessible in the devtools?
I don't know to which extent this is workable and useful. I'm confident 
it could be enough, but only user-research could really say.


### Promises and progress

Since I started talking about promises, I've had discussions with people 
and one thing that came about a couple of times was the idea of 
"progress" or how promises relate to streams.

The way I see promises, they have 3 states: unfulfilled/resolved/broken. 
I don't see an intermediate state. However, for some things (like data 
over the network or data pulled out of disk, or database records not 
coming all at once, etc.), one needs to consider that there are 
intermediate states. I think this is not a promise anymore, but rather a 
stream.
As a developer, when needing to decide whether I'll use a promise or a 
stream, I just ask myself: "can I do anything useful with the partial 
result?" if yes, that's a stream, if no, that's a promise.


I think I've shared pretty much all my experience and thoughts on the 
topic. I feel that overall, the Q API is really good to work with 
promises and my opinion is that a standard promise feature should have 
the same features (I however don't care about the exact names of methods).

## John J Barton

[_Fri Nov 9 09:01:07 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026250.html)

I also used Q (a slightly older version) for several months before removing
it because of cost/benefit.


>```javascript
>var someData1P = query(q1);
>someData1P.then(function(data){
>  // do something with data
>});
>someData1P.fail(function(error){
>  // handle the error
>});
>```

Note these callbacks: promise based code is very callback-y.

I agree that `Q.all` is the most valuable feature and the one that got me to
try Q and stick with it for a while. However, in my experience  this
feature is only needed a few times in an application.

> ### Debugging
> It's been said in other messages, one part where Q promises fell short was
> debugging.

I suppose if we stick to console logging I could agree with this very rosy
description. But Q makes breakpoint debugging essentially useless: the call
stack is gone and you have to break much more often then try to correlate
the values across breakpoints.

I agree that Q is exceptionally well done, so much so that I came away much
less enthusiastic about the promise concept overall.

## Mark S. Miller

[_Fri Nov 9 09:35:39 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026253.html)


>> Hi,
>>
>> In this message, I'll be sharing some experience I've had with the Q
>> library. I have worked with it for about 7-8 months in a medium/big Node.js
>> application (closed source, so I can't link, sorry).
>> I'll be covering only the parts that I have used during this experience.
>> It's related to my own writing style and I don't mean that the rest is
>> useless and should be thrown away, but the subset I'll be covering here has
>> proven to be sufficient to my needs for several months.
>>
>> I would be interested if others could share their experience if they had
>> a different way of using promises.
>>
>
> I also used Q (a slightly older version) for several months before
> removing it because of cost/benefit.

I would like to understand this better. When you using only the `.then`/`.when`
callback style, or were you using the `.send`/`.post` style to send eventual
messages to the promised object? In my own code, I use mostly the
`.send`/`.post` style, and I find it substantially improves readability.

See https://docs.google.com/file/d/0Bw0VXJKBgYPMU1gzQ3hkY0Vrbmc/edit
(with improved slides [here](http://code.google.com/p/es-lab/downloads/detail?name=friam.pdf#makechanges))
for a particularly elegant example using promises. I would be curious how
well you could express this without promises.


## John J Barton

[_Fri Nov 9 10:08:26 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026254.html)

Only `.then`/`.when`. That is the style that many examples promote. I don't
know about `.send`/`.post`.  (Style is also important in callback based async
coding, which is why the case against that approach is often over stated).

I also encountered a couple of cases where promises work exceptionally
well. If promises were built into the language and debuggers were upgraded,
then I would choose to use them for just these cases. Given both
preconditions I'm unsure I would use promises more broadly: they are not
the panacea I imagined when reading examples written by advocates.  I would
strongly advocate realistic evaluation of a debugging system for promises
(like eg Causeway) before committing to implementing them in the language.

## David Bruant

[_Tue Nov 13 02:35:13 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026315.html)

>```javascript
>function query(myQuery){
>  var def = Q.defer();
>
>  db.query(myQuery, function(err, data){
>    if(err)
>      def.reject(err)
>    else
>      def.resolve(data)
>    })
>
>  return def.promise;
>}
>```

I had lots of code like that.  If a library has a callback style, this boilerplate is 
necessary, but if a library naturally has promises, it's fine.
The good news is that with only a bit of boilerplate code, one can work 
with one or the other style. Things would be better, but could also be 
much worse.


> I suppose if we stick to console logging I could agree with this very 
> rosy description. But Q makes breakpoint debugging essentially 
> useless: the call stack is gone and you have to break much more often 
> then try to correlate the values across breakpoints.

True. You mentioned it in your last response, but built-in promises 
debugging would be of great help, because it has knowledge of the 
promise chain and could enable stepping from one resolution function to 
another resolution function.
Note that it would be impossible or close to impossible to have such a 
feature with DOMRequest (since they do not form a chain)

## Claus Reinke

[_Fri Nov 9 09:06:54 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026251.html)

> Both then and fail return another promise for the result of the callback
>
>```javascript
>var someData1P = query(q1);
>var processedDataP = someData1P.then(function first(data){
>    return process(data); // if what's returned here is a promise P, then
>         // processedDataP will be a promise for the resolution value of P, not
>         // a promise for a promise of the resolution value of P
>         // Said another way, in a .then callback, the argument is always a non-promise value
>  });
>
>var processedAgain = processedDataP.then(function second(processedData){
>    // play with processed data
>  });
>```

Perhaps I'm misunderstanding - my reading of the spec (fairly
concise tutorial at https://github.com/kriskowal/q) is: 

- originally, 'then's success callback returned values only, which 
    were wrapped into a promise by 'then'; 

- nowadays, 'then's success callback may also return a promise,
    which will *become* the result of 'then';

the latter option gives the type -if I read '.then' as a binary op
and focus on the success callback only-

    then : Promise(val) => (val => Promise(val2)) => Promise(val2)

which is familiar from monadic programming.

If I read this correctly, you could pass a promise to the next 'then'
callback by returning a promise that resolves to a promise (val2 
would itself be a promise;-).
 
> The fact that the .then of the next promise is called when the previous 
> normally returned and the .fail when the previous threw makes me feel 
> like promise chains have sort of two parallel channels to which one 
> decides to branch to by returning or throwing.

Yes, and that makes me a bit uneasy. The alternative style, with
only success callback for 'then' and only failure callback for 'fail'
makes more sense to me. It makes both error handling and error
passing more explicit - fail is the async catch.

One example of why mixing both callbacks into 'then' is misleading: 
the q-tutorial gives two examples of chaining styles, nested and flat. 

Refactoring flat chains into nested ones can be necessary when
access to multiple promise results is needed. That refactoring fails
if 'then's are used for failure callbacks, and using 'fail' would make
that more obvious.
 
> ### Debugging
> It's been said in other messages, one part where Q promises fell short 
> was debugging. With thrown errors, if you uncatch one, your 
> devtools/console will tell you. To my experience, with the Q library, if 
> you forget a .fail, an error may end up being forgotten which is bad 
> news in development mode (and I've wasted a lot of times not knowing an 
> error had happened and chasing this errors after understanding that's 
> why "nothing" was happening). I'm hopeful built-in promises will be able 
> to compensate.

Tooling for asynchronous programming is interesting in general.

As a minor helper: some debuggers allow to stop at any raised exception,
so you could see where it gets transformed into a failed promise. Another
idea: log such transformed errors, or automate matching of handled
failed promises against generated failed promises.
 
There is also the interesting issue of asynch coding constructing code
that will raise errors where the original error handlers wrapping the
code construction site aren't in place anymore.

## Domenic Denicola

[_Fri Nov 9 09:25:41 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026252.html)

> Another idea: log such transformed errors, or automate matching of handled failed promises against generated failed promises.

What we really need, as mentioned by Kris Kowal on Twitter recently, is the ability to *un*-console.log something. That is, we want to log unhandled rejections for the duration of them being unhandled, but if someone does come along and handle it, we need to get it out of the console.

Currently we kind of do this by exploiting a trick in some browsers' array-logging capabilities. If you log an empty array, but later modify its contents, it live-updates in Chrome (although this behavior might be going away from what I can tell) and I believe other browsers. This is a bit hacky and unpredictably supported.

All this has led us to contemplate building browser extensions with appropriate hooks for such logging. That's still an as-our-scarce-time-permits project, however.

## Claus Reinke

[_Fri Nov 9 13:57:09 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026263.html)

>> Another idea: log such transformed errors, or automate matching of handled failed promises 
>> against generated failed promises.
>
> What we really need, as mentioned by Kris Kowal on Twitter recently, is the ability to 
> *un*-console.log something. That is, we want to log unhandled rejections for the duration of them 
> being unhandled, but if someone does come along and handle it, we need to get it out of the 
> console.

yep, that was the rough direction, though I'd use an explicit off-console
log buffer (which might then be visualized continuously or pulled occasionally
for console snapshots, similar to heap profiles).

> Currently we kind of do this by exploiting a trick in some browsers' array-logging capabilities. 
> If you log an empty array, but later modify its contents, it live-updates in Chrome (although this 
> behavior might be going away from what I can tell) and I believe other browsers. This is a bit 
> hacky and unpredictably supported.
>
> All this has led us to contemplate building browser extensions with appropriate hooks for such 
> logging. That's still an as-our-scarce-time-permits project, however.

If your time is scarce and your goals involve generally valuable tooling,
why not explain your needs on js-tools? I can't promise that there will
be takers for every project, but encouraging sharing of efforts was one
of the goals for establishing that mailing list.

Same holds for sharing debugging war stories from which needs and
opportunities for better tooling might be inferred: don't hide them here
were they aren't actionable (there was an example of memory leak
tracking recently on this list, iirc).

## Brendan Eich

[_Fri Nov 9 18:14:03 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026264.html)

David Bruant wrote:
> Personally, to synchronize different async operations, I've never read 
> code more elegant than what Q.all offers.

What about task.js's join?

https://github.com/mozilla/task.js/blob/master/examples/read.html#L41

Generators + promises = tasks ;-)

## David Bruant

[_Tue Nov 13 02:43:53 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026316.html)

Le 10/11/2012 03:14, Brendan Eich a &#233;crit :
> David Bruant wrote:
>> Personally, to synchronize different async operations, I've never 
>> read code more elegant than what Q.all offers.
>
> What about task.js's join?
>
> https://github.com/mozilla/task.js/blob/master/examples/read.html#L41

I feel it's pretty much equivalent. Maybe slightly less verbose. I'd 
write the same code with promises as:

     Q.all(read("sleep.html"), read("read.html")).then(function(f1, f2){
         out.innerHTML += "sleep.html: " + (f1.responseText.length) + "\n";
         out.innerHTML += "read.html: " + (f2.responseText.length) + "\n";
     });

> Generators + promises = tasks ;-)

It took me several months to understand the value of tasks.js and then I 
loved the idea (though I haven't used it because of the lack of 
generators in platforms). The code you linked to leaves me somehow 
uneasy, because it looks like sync code while it's async. Promises have 
this advantage that they make clear what's sync and what's async. But 
maybe I also need to step out of my comfort zone for this case...

What's the error forwarding/handling story for tasks?

## Aymeric Vitte

[_Wed Nov 14 08:17:50 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026343.html)

I don't know when tasks.js will be "usable" (ie generators supported) 
but probably it's a very good solution to solve all the async (and 
resync) issues I had doing https://github.com/Ayms/node-Tor where 
unexpected and unpredictable things as well as performances reasons 
forced me to use [do_not_wait, setTimeout, clearTimeout,...] which makes 
part of the code look like a mess and that I found not easy at all to 
synchronize.

## Brendan Eich

[_Wed Nov 14 10:23:53 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026358.html)

David Bruant wrote:
> What's the error forwarding/handling story for tasks? 

That's the real beauty: try-catch and yield with thrown exceptions from 
resumed generators compose just as you'd expect.

## Mark S. Miller

[_Fri Nov 9 08:33:13 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026249.html)

Hi David, thanks for your thoughtful post. I've always used the two-arg
form of .then[1], but your post makes a strong case for more often using
separate one-arg `.then` and `.fail` calls. I say only "more often" because the
two arg form can easily make distinctions that the one-arg forms cannot

```javascript
var p2 = Q(p1).then(val => { throw foo(val); },
                    reason => { return bar(reason); });
```

is different than either of the one-arg chainings. It'll be interesting to
look over old code and see how often this difference matters. My guess is
it usually doesn't, in which case your style should dominate.

[1] In my code and in my Q specs <http://wiki.ecmascript.org/doku.php?id=strawman:concurrency> and
implementations <http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/ses/makeQ.js>,
I have been using "when" rather than "then". Because these are otherwise
compatible AFAICT with A+, for the sake of consensus I'm willing to change
this to "then" everywhere. But before I do, I'd like to make one last plea
for "when" and see how this community responds.

The word "when" is clearly temporal, and suggests postponing something
until some enabling condition. This seems perfect. The word "then" in
programming is most closely associated with the concept of an "if then
else", even though curly bracket languages never spell out the "then". When
I look at your `.then`/`.fail` examples, the first thought that always pops
into my head is "Shouldn't the opposite of `.then` be `.else` ?" Of course
`.fail` is appropriate and `.else` is not. But isn't `.then` inappropriate for
the same reason?

`.then` gets especially confusing when dealing with a promise for a boolean.
Consider the asyncAnd operation:

```javascript
function asyncAnd(answerPs) {
  let countDown = answerPs.length;
  if (countDown === 0) { return Q(true); }
  let {resultP, reject, resolve} = Q.defer();
  for (let answerP of answerPs) {
    Q(answerP).then(answer => {
      if (answer) {
        if (--countDown <= 0) { resolve(true); }
      } else {
        resolve(false);
      }
    }).fail(reason => { reject(reason); });
  }
  return resultP;
}
```


The code above, for good reason, uses both `then` and `else` in talking
about the same abstract value. This suggests the boolean reading for the
`then` that is just completely wrong. Replacing the `then` with a `when`
makes this code read well IMO.

If this argument fails to persuade us on es-discuss to switch to "when", I
will proceed to replace all my uses of `when` with `then` and declare this
terminology issue over.

Btw, above I tried using your one-arg `.then` and `.fail` style. I think this
worked well.


On Fri, Nov 9, 2012 at 4:33 AM, David Bruant <bruant.d at gmail.com> wrote:

> [...]
> #### Q.all
> My favorite feature is the Q.all function. Q.all accepts an array of
> promises and returns a promise which will be fulfilled when all promises
> are. The resolution values are the different promises resolution values:
>
>     var someData1P = query(q1);
>     var someData2P = query(q2);
>     var someData3P = fetch(url); // HTTP GET returning a promise
>
>     Q.all([someData1P, someData2P, someData3P])
>         .then(function(someData1, someData2, someData3){
>             // do something when all data are back
>         })
>
> I used this extensively and it's been extremely helpful. Personally, to
> synchronize different async operations, I've never read code more elegant
> than what Q.all offers. I'm interested in hearing what other's experience
> is on that point.
> Arguably, Q.all could take several arguments instead of accepting only an
> array (that's one thing I'd change). Maybe there is a good reason to
> enforce an array, but I don't know it.
>

I originally speced it to take varargs, but I was thinking in ES6 terms
where "..." is already supported. Kris Kowal points out that most usage of
Q today, for obvious reasons, is pre-ES6. The var-args form means that a
Q.all on a computed list would require usage of .apply, which is much
uglier than the extra square brackets for the non-computed case.

### Promises and progress
>
> Since I started talking about promises, I've had discussions with people
> and one thing that came about a couple of times was the idea of "progress"
> or how promises relate to streams.
>
> The way I see promises, they have 3 states: unfulfilled/resolved/broken.


Sigh. The fact that even you get confused on the terminology makes me think
that no state should ever be named "un" anything. Let's stick with the A+
terminology: pending/fulfilled/rejected.

History aside, I prefer "broken" to "rejected". (E used "broken") But the
verb form of "broken" is "break" which conflicts with a keyword. For this
reason, Tyler chose the verb "reject" for his original Q library, and this
choice stuck. This suggests that we name the state "rejected".

## Domenic Denicola

[_Fri Nov 9 11:32:35 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026255.html)

From: Mark S. Miller Friday, November 9, 2012 08:33

> I've always used the two-arg form of .then[1], but your post makes a strong case for more often using separate one-arg .then and .fail calls.

We have found this to be more expressive as well. Especially in ES5 environments, where we can use Q's alias of `catch` instead of `fail`:

```javascript
p1.then(val => doStuff)
     .catch(err => console.error(err));
```

> I have been using `when` rather than `then`. Because these are otherwise compatible AFAICT with A+, for the sake of consensus I'm willing to change this to `then` everywhere. But before I do, I'd like to make one last plea for `when` and see how this community responds.

I think perhaps because of my background as someone who has only ever programmed for nontrivial amounts of time in curly-bracket languages (C, C++, C#, JavaScript), I really don't see `then` as part of an "if-then-else" chain. None of your examples seem confusing to me! I don't know though, as this is obviously very subjective.

However, I see a lot of value in `when` as a word still. `Then` makes sense when used as a method:

```javascript
doThis().then(doThat).then(doAnotherThing)
```

But `when` makes sense when used as a function:

```javascript
let this = doThis();
let that = when(this, doThat);
let anotherThing = when(that, doAnotherThing);
```

or even

```javascript
let that = when(this).then(doThat);
```

where here `when()` is either making a value into a promise or assimilating an untrusted (or crappily-implemented) promise.

It also, to me, makes sense when used as a message, in the sense of `promiseSend`. This is a bit less important though I guess.

## Kevin Smith

[_Fri Nov 9 11:40:19 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026256.html)

> We have found this to be more expressive as well. Especially in ES5
> environments, where we can use Q's alias of `catch` instead of `fail`.

Nice: +1


> However, I see a lot of value in "when" as a word still. "Then" makes
> sense when used as a method:
>
> let that = when(this).then(doThat);
>

+1 here as well.

## Kevin Smith

[_Fri Nov 9 13:33:41 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026262.html)

>```javascript
>p1.then(val => doStuff)
>  .catch(err => console.error(err));
>```
>
> Nice: +1
>

On further thought, I'm not so sure.  Consider this code, which creates a
directory if it doesn't already exist and then logs "done".

```javascript
return AFS.stat(path).then(stat => {

    if (!stat.isDirectory())
        throw new Error("Path is not a directory.");

}, error => {

    // Path doesn't exist - create the directory
    return AFS.mkdir(path);

}).then(val => console.log("done"));
```


Breaking the error handler out into its own "fail" method seems to make
this awkward to express because the "fail" callback is assigned to the
"then" transform, instead of the original promise as intended.

I think arrow functions make the two-arg form considerably more
aesthetically pleasing.

## Brendan Eich

[_Fri Nov 9 18:47:42 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026265.html)

Kevin Smith wrote:
> I think arrow functions make the two-arg form considerably more 
> aesthetically pleasing.

Agreed. The single-parameter name can help avoid the "anonymous 
functions separated by comma" problem too.

## Claus Reinke

[_Sat Nov 10 07:05:40 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026268.html)

> On further thought, I'm not so sure.  Consider this code, which creates a
> directory if it doesn't already exist and then logs "done".
> 
>    return AFS.stat(path).then(stat => {
>        if (!stat.isDirectory())
>            throw new Error("Path is not a directory.");
>    }, error => {
>        // Path doesn't exist - create the directory
>        return AFS.mkdir(path);
>    }).then(val => console.log("done"));
> 
> Breaking the error handler out into its own "fail" method seems to make
> this awkward to express because the "fail" callback is assigned to the
> "then" transform, instead of the original promise as intended.

Conditional branching is as important for method chains as it is
for statement lists. But we do not use conditionals as the basic
statement form:

```javascript
if (true) { console.log(1) }
if (true) { console.log(2) }
```

One could also argue that promise failure is an exceptional condition,
so two-pronged 'then' encourages API designs that use exceptions for
normal branching - usually not a good sign.

> I think arrow functions make the two-arg form considerably more
> aesthetically pleasing.

Yes, but is that the only criterion? I'm not sure I can give a better API
on the spot, but there are some things that make me uneasy about
that sample. In particular, a recoverable condition (path doesn't exist)
and an unrecoverable error (path exists, is not a directory) are mixed
into the same channel (promise failure), which leads to the difficulty
you note (disentangling the conditions).

Ok, I had to try anyway;-) It took me a few attempts before I had an 
alternative that I was willing to show. How about this:

```javascript
return AFS.stat(path)
    .fail(error => null) // map failure to success condition
    .then(stat =>

        if (!stat)    // Path doesn't exist - create the directory
            return AFS.mkdir(path);
        else if (stat.isDirectory())    // done already
            return 'nothing to do';
        else    // no can do
            throw new Error("Path is not a directory.");

    ).then(val => console.log("done"));
```

We can then provide a standard promise method for the mapping 
from success/failure to nullable conditions, such as

    'cond' : cb => this.fail(error => null).then(cb)

and the code would become as pleasing as and the intentions 
arguably clearer than in the original example.

## Kevin Smith

[_Sat Nov 10 11:22:33 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026269.html)


Provided that fail provides the implicit success handler `val => val`, I
think that's correct.  Question:  is one-arg `then` + `fail` equally as
powerful as two-arg then?  Proof?

## Mark S. Miller

[_Sat Nov 10 15:46:07 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026270.html)

Is the following a counter-example?

>```javascript
>var p2 = Q(p1).then(val => { throw foo(val); },
>                    reason => { return bar(reason); });
>```
>
> is different than either of the one-arg chainings.
>

## Kevin Smith

[_Sun Nov 11 05:44:12 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026271.html)

I don't think so.  Consider the general two-arg form:

```javascript
promise.then(onSuccess, onFail);
```

We can transform that into a one-arg form like so:

```javascript
function WrappedError(err) {
    this.error = err;
}

promise
.fail(err => new WrappedError(err))
.then(val => val instanceof WrappedError ? onFail(val.error) : onSuccess(val));
```

But this involves the creation of an extra (unnecessary) node in the graph. And it's obtuse.  I still think the two-arg form makes the most sense as a base-level "then" API.

## Leo Meyerovich

[_Sun Nov 11 17:46:00 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026276.html)

> Many of the DOM APIs are effectively broken promises

Reading the thread again, is this really the primary motivation for adding promises? I don't see how the energy issue gets noticeably addressed, and so it sounds like, indeed, "you can always build a promise library on top of the current APIs." 

There may be real expressive value to a serious promise proposal, however. It can enable identifying asynchronous APIs and then 1) allowing frameworks to introspect on them and 2) clarifying the succeed/fail protocol. Doing so with standard promises seems awkward and insufficient, however, because their use only becomes apparent via duck typing: you wrap every API call, intercept the return, and check if it was a promise. Supporting introspection *before* the call seems more desirable. (So, perhaps, the proposal would be for a special async prototype added to any async API call).

## Brendan Eich

[_Sun Nov 11 22:37:32 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026277.html)

Have you checked out http://taskjs.org/ yet? Builds on ES6 generators 
(prefigured by ES4-era generators, inspired by Python 2.5 and with ES6 
matching PEP380 in intent and function if not form, and implemented in 
SpiderMonkey and Rhino).

## David Bruant

[_Tue Nov 13 02:27:35 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026314.html)

The underlying question is how to choose which should be the fundamental 
brick and which should be the library.
Historically, the choice was made following this schema:
* Some browser ships an feature with a given API
* Other browsers are forced the implement the same thing because soon 
enough, there is content relying on the given API.
Depending on the feature, there is variable amount of discussion among 
browsers and standard bodies before/between/after the 2 steps.
We know where this had led people writing JS applications: they write 
libraries to have other APIs. It wastes everyone's bandwidth (has a 
study been done of how much bandwidth is used per day just by jQuery 
downloads across the planet? It'd be fun to know) and has a runtime cost 
as well.
Since 2010-ish, JS devs fight back and improve APIs at the standard 
level where there is a worthwhile opportunity (I'm thinking about DOM4's 
Element.prototype.remove for instance).
TC39 is also eager to have more and more feedback from developers before 
freezing the spec.

Now, JS devs (well... and academia and a lot of other actors) didn't 
wait to find solutions to improve the async programming story. Promises 
is one outcome. As Alex pointed out in his response, promises have been 
adopted in a good share of JS client-side libraries and feel to have a 
good support from the JS dev community, somehow proving that promises 
are tha favored API to deal with async programming.

I feel there is a small window of opportunity to get promises as a 
standard features, so that's my motivation to talk about it.

## David Bruant

[_Tue Nov 13 06:33:59 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026318.html)

> But this involves the creation of an extra (unnecessary) node in the graph. And it's obtuse.  I still think the two-arg form makes the most sense as a base-level "then" API.

I agree. But I feel it's possible for engines to optimize (both in time and memory) by detecting the .then(func).fail() (or the other way around) pattern. Also, assuming async programming is related to DB query/disk access/network I think these costs are negligible when compared with the time to wait for network or a user input or a timeout.

If the second argument is optional, it's possible to have both one-arg and two-arg styles in the same API. What do people think about this idea?

## Kevin Smith

[_Wed Nov 14 08:23:07 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026344.html)

> If the second argument is optional, it's possible to have both one-arg and
> two-arg styles in the same API.
> What do people think about this idea?
>

Maybe - minimalism served the class proposal quite well.  It might be a
good strategy here, too.

Here's what I'm thinking:

```javascript
// Creates a new promise
let promise = new Promise();

// Resolves the promise (ala Q)
promise.resolve(value);

// Rejects the promise (ala Q)
promise.reject(value);

// A handle to the eventual value of the promise
promise.future;

// The then method (ala Promises/A+)
promise.future.then(val => {

    // Success handler

}, err => {

    // Error handler
});

// Returns a future for the value
Promise.when(value);

// Returns a rejected future with the specified error
Promise.reject(error);

// Returns a future for every eventual value in the list
Promise.whenAll(list);

// Returns a future for the first resolved future in the list
Promise.whenAny(list);
```

Initial implementation [here](https://github.com/jscloud/Promise)

I think it's important to separate "Promise" from "Future".  Back in the CommonJS mailing list days, there was contention between Promises/A (thenables) and Promises/B (basically Q).  But they really are complementary:  futures and promises.

## Mark S. Miller

[_Mon Nov 12 07:43:29 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026282.html)

On Fri, Nov 9, 2012 at 8:33 AM, Mark S. Miller wrote:

> [...] In my code and in my [Q specs](http://wiki.ecmascript.org/doku.php?id=strawman:concurrency) and
> [implementations](http://code.google.com/p/google-caja/source/browse/trunk/src/com/google/caja/ses/makeQ.js),
> I have been using "when" rather than "then". Because these are otherwise
> compatible AFAICT with A+, for the sake of consensus I'm willing to change
> this to "then" everywhere. But before I do, I'd like to make one last plea
> for "when" and see how this community responds.
> [...]

> If this argument fails to persuade us on es-discuss to switch to "when", I
> will proceed to replace all my uses of "when" with "then" and declare this
> terminology issue over.
>

The shift back to "when" clearly failed to achieve consensus. I just fixed
the wiki page to use "then" instead of "when". For consistency, I also
fixed it to use "there" instead of "where". This is only a bit awkward, as
"there" might be here but the word "there" soft implies somewhere other
than here. Since "when" has no other advocates, this terminology issue is
over.

Updates to code coming when I have time.

## Andreas Rossberg

[_Mon Nov 12 08:36:06 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026284.html)

> The shift back to "when" clearly failed to achieve consensus.

FWIW, I think "then" is better, because "when" sounds as if it should be passed some kind of predicate or condition. It just doesn't read very natural when taking continuations.

## David Bruant

[_Tue Nov 13 06:21:13 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026317.html)

09/11/2012 17:33, Mark S. Miller:

>```javascript
>var p2 = Q(p1).then(val => { throw foo(val); },
>                    reason => { return bar(reason); });
>```
>
> is different than either of the one-arg chainings....

Interesting example. Let's compare it with:

```javascript
     var p2 = p1.then(val => { throw foo(val); })
                .fail(reason => { return bar(reason); });
```

With this given order, the .fail catches the error in the .then callback. There is an information loss in the fact that in the fail callback, it may not be clear whether the error came from p1 or the .then callback. In my experience, it was never a worthwhile distinction (I just cared about "an error happened somewhere"), so I guess the 80% case is covered.

In case it really matters, the value thrown in the .then could be made distinguishable from expected thrown errors of `p1` so that the callback can compare and choose a different behavior for each case. Also, it's still possible to hook the `.fail` to `p1` directly if necessary.

I feel the one-arg version works fine. As you pointed out, there is a 
minor information loss, but it's not that hard to work around in the 
cases where it matters (I've seen you seem to agree, but I felt it was 
important to do the analysis of the differences in both cases).

### Terminology

I have to admit that I haven't paid close attention. For me, `.then` felt 
relevant as a form of punctuation like semi-colons for the synchronous 
style:

```javascript
p.then(doThis)
  .then(doThat)
  .then(...)

// akin to
var p2 = doThis(p);
var p3 = doThat(p2);
...
```

JavaScript is a bit special with semicolon, but in some other major languages, a semi-colon is a language construct to separate 2 steps in an algorithm and mean "do this then that". I used `.then` because of my experience with Q, but if "when" is more accurate, I'll be fine with when.

>> The way I see promises, they have 3 states:
>> unfulfilled/resolved/broken.
>
> Sigh. The fact that even you get confused on the terminology makes me 
> think that no state should ever be named "un" anything. Let's stick 
> with the A+ terminology: pending/fulfilled/rejected.

Sounds good to me. I'll pull my "non-native English speaker" card here, 
because I don't understand the difference between your terminology and 
the one I chose.

### Q.all

> I originally speced `Q.all` to take varargs, but I was thinking in ES6 
> terms where "..." is already supported. Kris Kowal points out that 
> most usage of Q today, for obvious reasons, is pre-ES6. The var-args 
> form means that a Q.all on a computed list would require usage of 
> .apply, which is much uglier than the extra square brackets for the 
> non-computed case.

That makes a lot of sense.
Assuming arrays and promises are unambiguously distinguishable, the 
Q.all feature could be polymorphic and take either one array as argument 
or a bunch or promises and have the expected behavior in each case.

## Leo Meyerovich

[_Sun Nov 11 13:02:37 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026272.html)

I've been lurking on this thread and am somewhat confused. Is the motivation for promises a needed expressive primitive, updating the standard library to have asynchronous calls and structure them via promises, sugar, or something else? I assume it's not to proscribe one particular style of coordination over another, which would have little empirical validation.

If it's for asynchronous standard API, why shouldn't coordination be a user-level framework decision? E.g., have one callback for each async API (or two, succeed and fail), and leave the rest up to the coordination framework of the day? Simple API structure, like one of these patterns, will help framework writers make generic wrappers, but introducing more seems excessive. The main use case for APIs would be (AFAICT) to allow frameworks to introspect on whether an API call is async. Promises would enable inspecting the return value, but this seems awkward and maybe even late.

If the motivation is for language-level issues, I'm curious as to how this relates to topics like compiler optimization, multiprocessing, cross-frame communication, etc. Furthermore, are there any semantic concerns coming up (e.g., E style vats), or is the intent here purely sugar? (I assume blocking on a Liskov-style future will not be allowed).

For sugar, a lightweight lambda and, if still desired, macros, seem to eliminate much of the need of a primitive. 

Hopefully my confusion as to the motivation and design space here is clear :)

## Alex Russell

[_Mon Nov 12 02:41:50 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026281.html)

I *really* don't want to wade deep into these waters, but I'll respond a bit here to correct errors:

Putting things into a language or standard library in no way "proscribes" alternatives; it does however anoint one variant as "idiomatic". To the extent a built-in does that, it should be following practice. Now, perhaps you haven't been watching the mis-designs in DOM or the ubiquitous promise/future/deferred-like libraries that are now part of every major JS toolkit on the client side (still our major user). Or perhaps you think that doesn't hold a lot of water as the language can go its own way (a view I'm *obviously* sympathetic to). But either way, I think Promises have value in that they:

 * were the basis for the underlying API we used to describe what happened when functions were de-sugared in the "await" proposal. They can do something similar for generators and, as someone who dislikes magic in general, I think we should do that work.
 * the weight of library authors sending down their own versions of this stuff is MASSIVE and *should* inform what we do. Any other bias needs massive justification. So yes, we have tons of validation on this. Just look around.

## Leo Meyerovich

[_Mon Nov 12 11:00:08 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026287.html)

Rearranging a bit:


>   * the weight of library authors sending down their own versions of this stuff is MASSIVE and *should* inform what we do. Any other bias needs massive justification. So yes, we have tons of validation on this. Just look around.


Q is 3KB compressed, and some of its primitives seemed unnecessary last time I looked. There are bigger frameworks, but they diverge, which suggests disagreement on more substantial coordination abstractions.


> To the extent a built-in does that, it should be following practice.


This assumes, as based on your comment above, that promises are not a performance construct. I largely agree for today's browser scenario.

Interestingly, a built-in may enable optimizations that a library cannot -- I'm curious as to the Node community's take. They've done some cool benchmarking, and I can see extending the promise API to be more data-centric leading to significant speedups. Despite my enthusiasm, I suspect the Node scenario merits more experimentation with high-performance implementations, not just libraries.

## Domenic Denicola

[_Wed Nov 14 08:25:48 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026345.html)

Kevin Smith:
> I think it's important to separate "Promise" from "Future".

Why go purposefully against the existing terminology of the JavaScript ecosystem? Just say "deferred" where you have "promise" and "promise" where you have "future" and you avoid needless confusion and conflict.

This isn't Scala; we have existing terminology for exactly these concepts. Just use it.

## Kevin Smith

[_Wed Nov 14 08:41:52 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026346.html)

I prefer to find the optimal solution first and then consider migration
costs later.  I think migration costs are in general overstated in these
matters (I made that mistake a couple of years ago with modules).

Plus, "Deferred", seriously?  It's not even a noun.   : )

## Domenic Denicola

[_Wed Nov 14 08:44:15 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026347.html)

I think you meant "optimally colored bikeshed," but OK.

## Kevin Smith

[_Wed Nov 14 09:25:13 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026352.html)

Ouch : )

Names are important.  Especially when it comes to something as potentially
confusing as promises and futures.

## Mark S. Miller

[_Wed Nov 14 09:41:28 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026354.html)

I agree that names are important.

1. First choice would be to use existing words, where their existing
meanings (whether nat-lang or prior technical use) clearly suggests their
intended technical meaning.
2. Second choice would be to use words whose existing meanings do not
conflict with their intended technical meaning, and where programmers do
not mistakenly assume they know the wrong meaning.
3. Like #2, but coining new words if needed.
4. ...anything else is too likely to cause confusion.

In doing the E non-blocking promises, I looked at the history of usage of
the terms "promise" and "future". Several people had tried to use both
terms in order to make a distinction. But historically, I could not find
any consistency. For every interesting distinction, "promise" and "future"
were used by someone on each side of the distinction. However, both were
used primarily for the abstraction of designating a yet-to-be-determined
value.

Historically, the primary examples I can think of [1] of an abstraction that
bundles both the delayed designation and the ability to determine what is
designated is the logic variable (especially in concurrent logic languages)
and the Python Deferred, which inspired various JS Defferds. If you
consider this prior history of Deferreds to be significant, then our
choices of Promise/Resolver/Deferred would be following strategy #1.
Otherwise, Deferred is a new coinage and fits in strategy #3. Either way,
Scala's unfortunate choice clearly violates this history in a confusing
manner, so I'd classify it as #4. Let's not repeat Scala's mistake.


[1] Off the top of my head now, without actually going back to the history.

## Andreas Rossberg

[_Wed Nov 14 09:57:58 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026356.html)

Just to reiterate, it's not just Scala, but more importantly also C++,
Java (to some extent), and several less mainstream languages. That is,
this use of terminology has quite a bit of history of its own, dating
back almost as far as E (and having developed more or less
independently).

## Kevin Smith

[_Wed Nov 14 10:46:33 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026359.html)

I personally think Scala made a good choice, because most of the time all
we are concerned with is the future.  And we can talk about futures
independently from invoking the promise analogy.  I think the promise
analogy is great, but in my opinion it can be a little bit tricky for
people to pick up.

If I were teaching this stuff, I wouldn't start with promises.  I would
start with futures and show how they are like callbacks++.  That's what
most users are looking for to begin with anyway.  Only after we'd mastered
futures would we talk about promises and their three states and remote
objects and so-on.

Also, something like a DOMFuture can be spec'd independently from any
particular (and more fully featured) promise spec.

## Tom Van Cutsem

[_Wed Nov 14 11:37:33 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026360.html)

I still think futures connote strongly with blocking synchronization. If
we'd add a concept named "future" to JS on the grounds that the same
concept exists in Java and C++, developers will reasonably expect a
blocking future.get() method.

In my experience, the term "promise" is much more associated with
non-blocking synchronization through .then or .when callback chaining
(although ironically the name derives from Argus, which featured blocking
promises. Argh! :-)

## Mark S. Miller

[_Wed Nov 14 12:11:00 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026362.html)

OTOH, the Argus promises were where [promise pipelining](http://en.wikipedia.org/wiki/Futures_and_promises#Promise_pipelining) was invented, and this is a key feature of our distributed promises via `.send`/`.post`. In this way, the Argus promises are the closest to ours.

## Rick Waldron

[_Wed Nov 14 09:28:29 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026353.html)

On Wed, Nov 14, 2012 at 11:25 AM, Domenic Denicola wrote:

>  Why go purposefully against the existing terminology of the JavaScript
> ecosystem?

It's true that the terminology exists in JS, but it's been identified that
these terms may have been misappropriated. Kevin's proposal is easier to
reason about:

"Promise to deliver a value in the Future"

vs.

"Defer the delivery of a value, and that's a Promise"

...awkward? Note that I disagree with jQuery's use of the terminology,
which matches the latter.




> This isn't Scala; we have existing terminology for exactly these concepts.
> Just use it.

I'd be interested in seeing usership data for libraries using this existing
terminology.

## Mark S. Miller

[_Wed Nov 14 09:48:22 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026355.html)

> It's true that the terminology exists in JS, but it's been identified that
> these terms may have been misappropriated.

"misappropriated"? What do you mean?

> Kevin's proposal is easier to reason about:
>
> "Promise to deliver a value in the Future"

This would make "promise" a verb, which is clearly its dominant nat-lang
use. However, I don't see how that justifies using it as the name for the
Deferred abstraction. "in the Future" uses future to name the time when the
value will be delivered. I don't see how this suggests anything appropriate
either.

For "Promise" as a noun, if I have a promise from you, I do not have the
ability to resolve the promise -- that ability is your's. So the ability of
resolve the promise is clearly distinct from having the promise.

## Rick Waldron

[_Wed Nov 14 10:13:35 PST 2012_](https://mail.mozilla.org/pipermail/es-discuss/2012-November/026357.html)


> "misappropriated"? What do you mean?

I was referring to the varied use of terminology illustrated in [Kevin's spreadsheet](https://docs.google.com/document/d/10OeEwqEuEPyDVRU9VXemxi3kc7ba_pugxHLD2BSrG_k/edit)

Originally I had worded the phrase something like "Make a Promise...";
regardless, this perspective clearly (and correctly) illustrates the
intention.