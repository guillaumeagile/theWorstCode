# Look at that simplistic code: it lies to you


```cs
public async Task<T?> Get(string uid)
        {
            var result = await _collection.FindAsync((x) => x.Uid == uid).Result.FirstAsync();
            return result;
        }
```

## analysis

the signature says: you will receive an async task of a T type that may be null (? marks a nullable value)

## why this can go wrong ?

if they is no content that match the predicate `(x.Uid == uid)`, you'll get an Exception`

```cs
System.InvalidOperationException: Sequence contains no elements
   at System.Linq.ThrowHelper.ThrowNoElementsException()
   at System.Linq.Enumerable.First[TSource](IEnumerable`1 source)
   at MongoDB.Driver.IAsyncCursorExtensions.FirstAsync[TDocument](IAsyncCursor`1 cursor, CancellationToken cancellationToken)
   at Gateways.MongoDB.MongoContentGatewayAsync`1.Get(String uid)
```

## what are the consequnces

Of course, as you don't do unit test , you go in production and that error pops.
In the most probable scenario, you (or the user) were asking to retreive a single value.
And it gets this `Sequence contains no elements` ... Quite puzzling.
Your brain is asking: wft?
You will have to open a log (takes time and heavy brain load), find out the message among others, as this message is very common in Linq.
When you'll find the correct one, you have to analyse the why. 
But the stack trace won"t give you more explanation, nor the Uid that you were looking for, and maybe see that the problem is malformed Uid maybe. Or a real lack of data in the base.
 Your brain will have to do extra efforts to

 ## how to solve it, first step

To avoid that annoying exception of `Sequence contains no elements`you can add something that turn 'nothing' into a default value, like:

```cs
public async Task<T?> Get(string uid)
        {
            var result = await _collection.FindAsync((x) => x.Uid == uid).Result.FirstOrDefaultAsync();
            return result;
        }
```

but it's not a big step, you just transform a problem into another:  NULL POINTER EXCEPTION, that will explode in your face, later in the code.

 Instead of using
