
## Property based testing


C# and Java code can be unit tested, as can F# code. The Xunit test framework allows different sets of parameters to be passed to unit tests using the [Theory] and [InlineData(99)] attributes, effectively creating a different unit test for each data set.

```C#
[Theory]
[InlineData(3)]
[InlineData(5)]
[InlineData(6)]
public void MyFirstTheory(int value)
{
    Assert.True(IsOdd(value));
}
```

Property testing is where this idea is extended to allow the test framework to automatically generate test parameters, instead of them being supplied by the developer via [InlineData(xx)] attributes. This example uses has simple 'int' parameter, but automatic generation of arbitrarily complex types works just as well. Each test can be run a configurable number of times, the default is 100 but it can be set to millions. 

In the test below random but valid DerivativeSecurityListRequest FIX messages are serialised and deserialised, and the output compared with the input.

```F#
[<PropTest>]
let WriteReadRoundTrip_DerivativeSecurityListRequest (msgIn:Fix44.Messages.DerivativeSecurityListRequest) = 
    let bs        = Array.zeroCreate<byte> bufSize
    let posW      = Fix44.MsgWriters.WriteDerivativeSecurityListRequest bs 0 msgIn          // writing msgIn to a buffer
    let index     = Array.zeroCreate<FIXBufIndexer.FieldPos> indexBufSize
    let indexEnd  = FIXBufIndexer.BuildIndex index bs posW
    let indexData = FIXBufIndexer.IndexData(indexEnd, index)
    let msgOut    = Fix44.MsgReaders.ReadDerivativeSecurityListRequest bs IndexData        // reading the buffer to get msgOut
    msgIn         = msgOut                                                                 // is msgIn the same as msgOut, F# uses structural equality by default
```

The property test framework used by F# is [FsCheck](https://fscheck.github.io/FsCheck), there are good examples of using property testing in [F# for Fun and Profit](http://fsharpforfunandprofit.com/posts/property-based-testing).

When FsCheck finds a test failure it searches for a simpler failing test. In the case of the 'int' example simpler would mean a smaller int, then FsCheck displays the simplest version of the failing test. This is called 'shrinking'.

If the type being generated cannot represent invalid states, and FsCheck runs the test a very large number of times then there will be more confidence in your code then "confidence = numTestRuns / numValidStates". No one claims this will find all bugs, but it is likely to find bugs that would not be found with other techniques.

Unit tests are still useful, for instance if you want to test that your parsing code can handle invalid input, in FsFIX this is used to test that fields that should not be in a buffer for a particular message type are detected.

Tests like WriteReadRoundTrip_DerivativeSecurityListRequest can be run in the Visual Studio test explorer or by using TestDriven.net. FsCheck integrates with Xunit to enable this. 

## QuickFix(N|J) Echo

FsChecks FIX message generation can be bent to other purposes, such as generating FIX messages in FsFIX, sending the messages to some other FIX engine which is modified to deserialise into a FIX object, then serialise and return that object to FsFix. QuickFixN and QuickFixJ (C# and Java open source FIX engines) both come with 'Executor' demo apps, these were modified to do this. The client is the FsFIXEcho app (which is in), which had just enough FIX session logic to be able to logon to the modified QuickFix executors, and a property test taking a FIX message parameter. 




* the [<PropTest>] attribute does not come with FsCheck but is easy to define, 
















 using FsFIX echo with QuickfixN and QuickfixJ 












### FsFix issues (now resolved)

FsFIX originally assummed that all fields are in the order they appear in the FIX xml spec. Correcting this required adding logic to search for fields in after indexing the byte buffer.


when a field appears more than once in a FIX message, e.g. once in the msg and othertimes in a group in the msg
if the first/outer instance of the field is optional then the value


### QuickFixN issues

1. DateTimes and Times involving [Leapseconds](https://en.wikipedia.org/wiki/Leap_second) are not recognised as valid
2. RawData fields containing field or tag-value seperators raise formatting errors

