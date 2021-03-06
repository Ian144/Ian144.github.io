# Property Based Testing

FsFIX uses property tests as well as unit tests. The property test framework used by FsFIX is [FsCheck](https://fscheck.github.io/FsCheck). Examples of using property testing can be found [here](http://fsharpforfunandprofit.com/posts/property-based-testing).

## What is property based testing?

Consider a progression from 'unit tests' to 'theory based unit tests with few sets of inline data' to 'property based tests with a large set randomized data'. F# code can be unit tested using the same tools as C#. The Xunit test framework allows different sets of parameters to be passed to unit tests using the [Theory] and [InlineData(99)] attributes, effectively creating a different unit test for each data set.

```C#
[Theory]
[InlineData('a')]
[InlineData('b')]
[InlineData('?')]
public void MyTheory(char value)
{
    Assert.True( Char.IsLetter(value) );
}
```

Property testing extends this idea. Instead of test data being supplied by the developer via [InlineData(xx)] attributes, the test framework automatically generates test data. Automatic generation of instances for arbitrarily complex types works just as well as for primitive types. Each property test can be run a configurable number of times; the default is 100 but it can be set to millions (which will take a long time to run). In a sense, running a property test is running a search for undiscovered bugs and incorrect assumptions.

In the FsFIX test below, random but valid DerivativeSecurityListRequest FIX messages are serialized and deserialized, then the output is compared with the input.

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

If the type being generated cannot represent invalid states, such as when the type is an ADT, and FsCheck runs the test a very large number of times then it becomes more likely that the code under test satisfies the test property. No one claims this will find all bugs, but it is likely to find bugs that would not be found with other techniques. Property testing is not a silver bullet, but it is a useful and underused (outside of functional programming) tool.

Unit tests are still useful, for instance if you want to test that your parsing code can handle invalid input. In FsFIX this is used to test that fields that should not be in a buffer for a particular message type are detected.

Tests like WriteReadRoundTrip_DerivativeSecurityListRequest can be run in the Visual Studio test explorer or by using TestDriven.net. FsCheck integrates with Xunit to enable this.


## Using property testing to send random FIX messages to QuickFixJ and QuickFixN

FsCheck type instance generation can be bent to other purposes, such as generating random but valid FIX messages in FsFIXEcho, which sends the messages to some other FIX engine application, coded to return the message it received after deserializing and reserializing it. QuickFixN and QuickFixJ (C# and Java open source FIX engines) both come with 'Executor' demo projects, which were modified to do just that. FsFIXEcho has just enough FIX session logic to be able to login to the modified QuickFix executors.  FsFIXEcho runs a property test that sends messages to QuickFix and checks that the  message received in reply  is the same as the outgoing message.

FsFIXCodeGen generates F# code from a FIX44.xml spec, but there is more than one version of this file; QuickFixJ's version differs slightly from QuickFixN's. To test against QuickFiXJ first run FsFIXCodeGen against the version of FIX44.xml that comes with QuickFiXJ (pass the path to FsFIXCodeGen as a command-line parameter). 


## Issues found by FsFIXEcho with FsFIX, QuickFIXJ, QuickFIXN and the FIX 4.4 spec


### FsFix issues (now resolved)

1. FsFIX originally assumed that all fields in a FIX message appear in the buffer in the same order they appear in the FIX XML spec, outside of repeating groups this assumption was incorrect, QuickFixN and QuickFixJ do not work this way.

2. After fixing the issue above, FsFIX was unable to handle scenarios when an optional field can appear in the message itself and again inside a repeating group inside the message. Specifically the SettlementInstructions message contains an optional SettlInstSource field, but this field also appears inside a repeating group inside the SettlementInstructions message.


### FIX 4.4 issues

The MiscFeeType and MassCancelRejectReason fields are defined by the FIX spec as Char fields, but MiscFeeType.Agent, MiscFeeType.CONVERSION, MiscFeeType.PER_TRANSACTION and MassCancelRejectReason.OTHER have two character values. Such field values did not create test failures when running FsFIX property tests, it did show up when testing FsFIXEcho against QuickFixN, which understandably assumed Char fields should contribute 1 to the length. This looks like an issue with FIX4.4, these fields are of type 'String' in FIX5.0.

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

2. RawData fields containing field or tag-value separators raise formatting errors. RawData fields are allowed by the FIX spec to contain separators. Both QuickFixJ and FsFIX handle RawData fields correctly.

### QuickFixJ possible issue

FsFIXEcho did not find any issues with QuickFixJ, although the BusinessMessageReject message did cause QuickFixJ executor_echo process to appear to hang, maybe because BusinessMessageReject is an 'admin like' message, and QuickFixJ session logic expects the rejection to refer to an earlier message.



## Summary

ADTs combined with property based tests can help find errors that would otherwise be missed. The author has not previously worked with FIX, but by using property based testing found errors in the FIX spec not noticed when the specification was released.



