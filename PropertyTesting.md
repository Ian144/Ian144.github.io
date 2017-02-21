
## Property based testing


F# code can be unit tested using the same tools as C#. The Xunit test framework allows different sets of parameters to be passed to unit tests using the [Theory] and [InlineData(99)] attributes, effectively creating a different unit test for each data set.

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

Property testing extends this idea, the test framework to automatically generates test parameters, instead of them being supplied by the developer via [InlineData(xx)] attributes. This example uses has simple 'int' parameter, but automatic generation of arbitrarily complex types works just as well. Each test can be run a configurable number of times, the default is 100 but it can be set to millions. In a sense, running a property test is running a search for undiscovered bugs, unit tests fulfill a different purpose.


In the test below random but valid DerivativeSecurityListRequest FIX messages are serialized and deserialized, and the output is compared with the input.

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

The property test framework used by F# is [FsCheck](https://fscheck.github.io/FsCheck), good examples of using property testing can be found [here](http://fsharpforfunandprofit.com/posts/property-based-testing).

When FsCheck finds a test failure it searches for a simpler failing test. In the case of the 'int' example simpler would mean a smaller int, then FsCheck displays the simplest version of the failing test. This is called 'shrinking'.

If the type being generated cannot represent invalid states, and FsCheck runs the test a very large number of times then it becomes more likely that the code under test satisfies the test property: "code confidence = numTestRuns / numValidStates". No one claims this will find all bugs, but it is likely to find bugs that would not be found with other techniques. Property testing is not a silver bullet, but it is a useful and underused (outside of functional programming) tool.

Unit tests are still useful, for instance if you want to test that your parsing code can handle invalid input, in FsFIX this is used to test that fields that should not be in a buffer for a particular message type are detected.

Tests like WriteReadRoundTrip_DerivativeSecurityListRequest can be run in the Visual Studio test explorer or by using TestDriven.net. FsCheck integrates with Xunit to enable this.



## Using property testing to send random FIX messages to QuickFixJ and QuickFixN

FsCheck type instance generation can be bent to other purposes, such as generating FIX messages in FsFIXEcho which sends the messages to some other FIX engine application, coded to return the message it received after deserializing and reserializing it. QuickFixN and QuickFixJ (C# and Java open source FIX engines) both come with 'Executor' demo projects, these were modified to do just that. FsFIXEcho has just enough FIX session logic to be able to login to the modified QuickFix(J|N)executors, and runs a property test that checks that the  message that has done the full round-trip is the same as the original outgoing message.

FsFIXCodeGen generates F# code from a FIX44.xml spec, but there is more than one version of this file, QuickFixJ's version differs slightly from QuickFixN's. To test against QuickFiXJ first run FsFIXCodeGen against the version of FIX44.xml that comes with QuickFiXJ (pass the path to FsFIXCodeGen as a command-line parameter). 

This 'echo' testing found issues with FsFIX, 



### FsFix issues (now resolved)

1. FsFIX originally assumed that all fields are in the order they appear in the FIX XML spec, this assumption was incorrect, and QuickFixN and QuickFixJ do not work this way, messages sent to either can be echo'd back with the fields in a different order.

2. After fixing the issue above FsFIX was confused when an optional field can appear in the message itself, and again inside repeating group inside the message. Specifically the SettlementInstructions message contains an optional SettlInstSource field, but this field also appears inside a repeating group inside the SettlementInstructions message.


### FIX spec issues

The MiscFeeType and MassCancelRejectReason fields are defined by the FIX spec as Char fields, meaning they can be of any single character. But MiscFeeType.Agent, MiscFeeType.CONVERSION, MiscFeeType.PER_TRANSACTION and MassCancelRejectReason.OTHER have two character values. This did not flag errors when running FsFIX property tests, it did show up when testing FsFIXEcho against QuickFixN, which understandably assumed Char fields should contribute 1 to the length. This looks like an issue with FIX4.4, as these fields are of type 'String' in FIX5.0.

```xml
<field name="MiscFeeType" number="139" type="CHAR">
    <value description="AGENT" enum="12"/>
    <value description="CONSUMPTION_TAX" enum="9"/>
    <value description="CONVERSION" enum="11"/>
    <value description="EXCHANGE_FEES" enum="4"/>
    <value description="LEVY" enum="6"/>
    <value description="LOCAL_COMMISSION" enum="3"/>
    <value description="MARKUP" enum="8"/>
    <value description="OTHER" enum="7"/>
    <value description="PER_TRANSACTION" enum="10"/>
    <value description="REGULATORY" enum="1"/>
    <value description="STAMP" enum="5"/>
    <value description="TAX" enum="2"/>

<field name="MassCancelRejectReason" number="532" type="CHAR">
    <value description="INVALID_OR_UNKNOWN_CFICODE" enum="4"/>
    <value description="INVALID_OR_UNKNOWN_PRODUCT" enum="3"/>
    <value description="INVALID_OR_UNKNOWN_SECURITY_TYPE" enum="5"/>
    <value description="INVALID_OR_UNKNOWN_SECURITY" enum="1"/>
    <value description="INVALID_OR_UNKNOWN_TRADING_SESSION" enum="6"/>
    <value description="INVALID_OR_UNKNOWN_UNDERLYING" enum="2"/>
    <value description="MASS_CANCEL_NOT_SUPPORTED" enum="0"/>
    <value description="OTHER" enum="99"/>
```

FsFIX treats these fields as multi-case discriminated unions, and pattern matches against the byte representation of the 'enum' value for each case irrespective of the number of bytes, and so did not have problems reading and writing these fields.


### QuickFixN issues

1. DateTimes and Times involving [leap-seconds](https://en.wikipedia.org/wiki/Leap_second) are not recognized as valid

2. RawData fields containing field or tag-value separators raise formatting errors. RawData fields are allowed by the FIX spec to contain separators, both QuickFixJ and FsFIX handle RawData fields correctly.

### QuickFixJ possible issue

FsFIXEcho did not find any issues with QuickFixJ, although the BusinessMessageReject message did cause QuickFixJH echo to appear to hang, maybe because BusinessMessageReject is an 'admin like' message, and QuickFixJ session logic expects the rejection to refer to an earlier message sent to FsFIXEcho.










