# Look at that simplistic code: it lies to you
and will cause a lot of unecessary pain!


```cs
public async Task<T?> Get(string uid)
        {
            var result = await _collection.FindAsync((x) => x.Uid == uid).Result.FirstAsync();
            return result;
        }
```

### PROS:
quick to write, anyone can understand?

### CONS
problems will arise very soon

## Analysis

the signature says: you will receive an async task of a T type that may be null (? marks a nullable value)

## Why this can go wrong ?

if they is no content that match the predicate `(x.Uid == uid)`, you'll get an Exception`

```cs
System.InvalidOperationException: Sequence contains no elements
   at System.Linq.ThrowHelper.ThrowNoElementsException()
   at System.Linq.Enumerable.First[TSource](IEnumerable`1 source)
   at MongoDB.Driver.IAsyncCursorExtensions.FirstAsync[TDocument](IAsyncCursor`1 cursor, CancellationToken cancellationToken)
   at Gateways.MongoDB.MongoContentGatewayAsync`1.Get(String uid)
```

## What are the conseqeunces

Of course, as you don't do unit test , you go in production and that error pops.
What will happen?
In the most probable scenario, you (or the user) were just asking to retreive a single value.
And it gets this `Sequence contains no elements` ... Quite puzzling, you didn't expect a sequence, just a value.
Your brain is asking: wtf?
You will have to open a log (takes time and heavy brain load), find out the message among others, as this message is very common in Linq.
When you'll find the correct one, you have to analyse the why. 
But the stack trace won"t give you more explanation, nor the Uid that you were looking for, and maybe you'll have to figure out that the problem is malformed Uid, who knows.
Or a real lack of data in the base.
 Your brain will have to do extra efforts to

 ## How to solve it, first step

To avoid that annoying exception of `Sequence contains no elements`you can add something that turn 'nothing' into a default value, like:

```cs
public async Task<T?> Get(string uid)
        {
            var result = await _collection.FindAsync((x) => x.Uid == uid).Result.FirstOrDefaultAsync();
            return result;
        }
```

but it's not a big step, you just transform a problem into another:  NULL POINTER EXCEPTION, that will explode in your face, later in the code.
The new problem is:  FirstOrDefault returns null as a Default.
It's a catastrophy.

### The question you have to reason about is this:
what could be an honnest Default Value for my objects?
Null (even from nullables) are broken code. THey don't produce value, they just make your program crash.

# Is there a life after NULL ?
Of course!!
First you can use the [NULL OBJECT PATTERN](https://sourcemaking.com/design_patterns/null_object)

Then, if you have a little courage, why don't use the [MayBe or Option Monads](https://functionalprogramming.medium.com/null-object-design-pattern-and-maybe-monad-in-c-5c83c3b58bd4) ?

I'll explain more about them later, in the easy way, affordable for everyone, without Maths (only Cats).

Meanwhile, here's a good read: (https://leanpub.com/composingsoftware)
