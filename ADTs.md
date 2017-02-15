# The Unreasonable Effectiveness of Algebraic Data Types


Structs and classes are pretty much all there is when it comes to designing your own types in object oriented programming languages. The ML branch of functional programming, which includes F#, has an alternative - Algebraic Data Types (ADTs). A good introduction to ADTs, why they are useful, and using them in F# is Scott Wlaschins NDC talk: [Domain modelling with the F# type system](https://vimeo.com/97507575), also useful is the [F# wikibook](https://en.wikibooks.org/wiki/F_Sharp_Programming/Discriminated_Unions). ADTs can be concise, facilitate [making invalid states unrepresentable](http://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/) and work well with property based testing, see [here](http://fsharpforfunandprofit.com/posts/property-based-testing) or [here](https://fscheck.github.io/FsCheck/QuickStart.html). Nulls are less of an issue in F# compared to C# and Java and variables are immutable by default. 

 FsFixGen F# projects can be found in GitHub at [https://github.com/Ian144/fsFixGen](https://github.com/Ian144/fsFixGen). FsFixGen creates ADTs to represent FIX messages, groups and fields from the same xml FIX specification used as source by the Java and C# versions of quickfix. FsFixGen also generates code to read and write FIX messages to and from byte arrays. The generated F# FIX code has been checked in (so you don't need to generate it yourself to view it), and can be found in the fsFix subproject. There are several different versions of FIX, FsFIXGen currently works for FIX 4.4, but should work with other versions with a little modification. 

## Compile time instead of runtime errors

Some FIX fields have a finite set of valid values, such as 'PosType', in quickFix/Java PosType this looks like

```Java
public class PosType extends StringField {

    static final long serialVersionUID = 20050617;

    public static final int FIELD = 703;
    public static final String TRANSACTION_QUANTITY = "TQ";
    public static final String INTRA_SPREAD_QTY = "IAS";
    public static final String INTER_SPREAD_QTY = "IES";
    public static final String END_OF_DAY_QTY = "FIN";
    public static final String START_OF_DAY_QTY = "SOD";
    public static final String OPTION_EXERCISE_QTY = "EX";
    public static final String OPTION_ASSIGNMENT = "AS";
    public static final String TRANSACTION_FROM_EXERCISE = "TX";
    public static final String TRANSACTION_FROM_ASSIGNMENT = "TA";
    public static final String PIT_TRADE_QTY = "PIT";
    public static final String TRANSFER_TRADE_QTY = "TRF";
    public static final String ELECTRONIC_TRADE_QTY = "ETR";
    public static final String ALLOCATION_TRADE_QTY = "ALC";
    public static final String ADJUSTMENT_QTY = "PA";
    public static final String AS_OF_TRADE_QTY = "ASF";
    public static final String DELIVERY_QTY = "DLV";
    public static final String TOTAL_TRANSACTION_QTY = "TOT";
    public static final String CROSS_MARGIN_QTY = "XM";
    public static final String INTEGRAL_SPLIT = "SPL";
    public static final String RECEIVE_QUANTITY = "RCV";
    public static final String CORPORATE_ACTION_ADJUSTMENT = "CAA";
    public static final String DELIVERY_NOTICE_QTY = "DN";
    public static final String EXCHANGE_FOR_PHYSICAL_QTY = "EP";
    
    public PosType() {
        super(703);
    }

    public PosType(String data) {
        super(703, data);
    }
}
```
The Java PosType class stores the case as a string, there are string constants defined, but their use is not type-checked, so the code below compiles.

```Java
quickfix.field.PosType pt = new quickfix.field.PosType("any old string");
```

In FsFix F# it is impossible to express such an error in compilable code, the F# PosType discriminated union can only take the set of values defined by the type. 

```fsharp
type PosType =
    | TransactionQuantity
    | IntraSpreadQty
    | InterSpreadQty
    | EndOfDayQty
    | StartOfDayQty
    | OptionExerciseQty
    | OptionAssignment
    | TransactionFromExercise
    | TransactionFromAssignment
    | PitTradeQty
    | TransferTradeQty
    | ElectronicTradeQty
    | AllocationTradeQty
    | AdjustmentQty
    | AsOfTradeQty
    | DeliveryQty
    | TotalTransactionQty
    | CrossMarginQty
    | IntegralSplit
```

### FsFIX merges length + data field pairs

FIX specifies some related pairs of fields, whereby one field contains data and the preceding field contains the length of the data. The data field may contain FIX field and tag-value separators, though these must not be treated as seperators. Quickfix defines both fields separately, FsFix elides the length field. Length is read from or written to the FIX buffer by 

```Java
public class RawDataLength extends IntField {
    static final long serialVersionUID = 20050617;
    public static final int FIELD = 95;

    public RawDataLength() {
        super(95);
    }

    public RawDataLength(int data) {
        super(95, data);
    }
}

public class RawData extends StringField {
    static final long serialVersionUID = 20050617;
    public static final int FIELD = 96;
    
    public RawData() {
        super(96);
    }

    public RawData(String data) {
        super(96, data);
    }
}

```

The same field in FsFIX F# is

```fsharp
type RawData =
    |RawData of NonEmptyByteArray.NonEmptyByteArray
     member x.Value = let (RawData v) = x in v
```

## Producing compilable F# from the xml FIX spec

- merging FIX length+data field pairs
- topological sorting of groups and components 
- merging groups with the same structure. FIX groups are defined in the messages or components which contain them, where possible FsFIX merges common group definitions into a single ADT group definition.
- the most common case is merged
  other possible merges are left with the name of their parent
  topographical sort of groups and components so they are generated in depency order, if groupX has a componentY member then the componentY ADT needs to be defined in source before group
OTHER RULES (take from F# code) HERE

## FsFIX features
## FsFIX characteristics
- FIX Components are represented as ADTs so work with intellisense, but have no effect in the byte stream representation of the fix msg. 
- Multicase FIX fields are represented by Multicase ADT discriminated unions (DUs)
- Single value Fix fields are represented by a single-case DU, not raw strings, ints etc
- Fields do not know their tag, field read/write functions do know their tag
- Optional fields, groups or components are wrapped in the F# Option type
- FIX message reading detects unexpected fields in FIX buffers
- Field reading copes with unordered msg body fields (unordered compared the order in the fix spec)
- Time and DateTime types cope with leap-seconds as per the FIX spec
- Group reading expects fields and sub-groups within the group to be ordered and sequential, but the group can start in any position
- Extra group instances are detected, if the number of groups is stated to be 4 but 5 are present then this will be detected
- Unexpected fields for a given msg type are detected
- Fields which may contain field or tag-value separators, such as RawData


## Codesize FsFIX vs QuickfixN

    NOTE TO SELF - THIS MAY NOT BE BEING FAIR TO QUICKFIX, as FsFIX WILL REQUIRE Make<MSGNAME> TO HAVE THE SAME CONVENIENCE WHICH MEANS ADDING 

    quickfixN message definitions: 14.4MB in 94 files
    quickfixN message definitions: 
    quickfixN field definitions: 647kb non comment bytes in 1 file (seems to be a single file for all FIX versions, 999 fields)

    Fix44.Messages.fs: 87.2kb
    FsFix field definitions FIX4.4 only: 117kb in 1 file (for FIX4.4 only 900 fields, 916 before len+data merge )


