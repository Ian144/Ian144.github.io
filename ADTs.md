# The Unreasonable Effectiveness of Algebraic Data Types


Structs and classes are pretty much all there is when it comes to designing your own types in object oriented programming languages. The ML branch of functional programming, which includes F#, has an alternative - Algebraic Data Types (ADTs). A good introduction to ADTs, why they are useful, and using them in F# is Scott Wlaschins NDC talk: [Domain modelling with the F# type system](https://vimeo.com/97507575), also useful is the [F# wikibook](https://en.wikibooks.org/wiki/F_Sharp_Programming/Discriminated_Unions). ADTs can be concise, facilitate [making invalid states unrepresentable](http://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/) and work well with property based testing, see [here](http://fsharpforfunandprofit.com/posts/property-based-testing) or [here](https://fscheck.github.io/FsCheck/QuickStart.html). I thought Algebraic Data Types (ADTs) would be a good match for representing FIX messages and that it would be interesting to try.

FsFixGen creates ADTs to represent FIX messages, groups and fields from the same xml FIX specification used as source by the Java and C# versions of QuickFix. FsFixGen also generates code to read and write FIX messages to and from byte arrays. The generated F# FIX code has been checked in (you don't need to generate it yourself to view it, although you can), and can be found in the FsFIX subproject. There are several different versions of FIX, FsFIXGen currently works for FIX 4.4, but should work with other versions with a little modification. 


## ADTs can move some runtime errors to compile time

### Some FIX fields have a finite set of valid values, such as 'PosType', in quickFix/Java PosType this looks like

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
The Java PosType class stores the case as a string. There are string constants defined, but their use is not type-checked, so the code below compiles.

```Java
quickfix.field.PosType pt = new quickfix.field.PosType("any old string");
```

In FsFix F# it is impossible to express such an error in compilable code. There are two main forms of ADT, discriminated unions and records. Discriminated unions have a finite set of cases, the discriminated union for PosType is


```F#
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

### Simple FIX fields are wrapped by a single-case discriminated union.

(by 'simple' I mean fields that do not have a finite set of valid cases, such as PosType)

Once wrapped, type checking will raise compilation errors if code attempts to assign an AdvID value to an Account field

```F#
type Account =
    |Account of string
     member x.Value = let (Account v) = x in v


type AdvId =
    |AdvId of string
     member x.Value = let (AdvId v) = x in v
```



### What FsFIX does not do for you (and a little bit more of what it does)

With one exception, FsFIX F# can only introduce constraints, and thereby compile time errors, if those constraints are expressed in the FIX XML spec (i.e. FIX4.4.xml), such as the set of legal values PosType can take. If different fields in the same message have constraints such that the value of one affects legal values of the other, possibly expressed in FIX documentation but not in FIX4.4.xml, then FsFIX cannot help you there. The exception is for pairs of fields where one field contains the length of the data in the other, e.g. RawDataLength and RawData. FsFIX types store the data in an array and elide the RawDataLength field. When it is time to write the message containing RawData, the value of RawDataLength is taken from the length of the array, it cannot be incorrectly set by buggy code elsewhere, see [merging length + data field pairs](GeneratingFsFIX.md).


## ADTs are concise

The FIX4.4.xml spec defines the UserRequest message like this

```Xml
<message name="UserRequest" msgtype="BE" msgcat="app">
    <field name="UserRequestID" required="Y" />
    <field name="UserRequestType" required="Y" />
    <field name="Username" required="Y" />
    <field name="Password" required="N" />
    <field name="NewPassword" required="N" />
    <field name="RawDataLength" required="N" />
    <field name="RawData" required="N" />
</message>
```

The generated F# ADT looks very similar and is of comparable length to the XML definition, and is easy to take-in and understand

```F#
type UserRequest = {
    UserRequestID: UserRequestID
    UserRequestType: UserRequestType
    Username: Username
    Password: Password option
    NewPassword: NewPassword option
    RawData: RawData option
    }
```

To be fair to QuickFixJ and QuickFixN, creating FsFIX messages that have many optional fields could be tedious. FIX message types have many optional fields, there are 119 in an ExecutionReport. Typing all the 'None's would be painful, so helper factory functions are generated like the one below for UserRequest. 

```F#
let MkUserRequest (userRequestID:UserRequestID, userRequestType:UserRequestType, username:Username) : UserRequest = {
    UserRequestID = userRequestID
    UserRequestType = userRequestType
    Username = username
    Password = None
    NewPassword = None
    RawData = None
  }
```

The QuickFix Java implementation of UserRequest, below, offers no more functionality than the F# version but is considerably longer, 4453 characters vs 492 for the FsFIX definition + factory function. 

```Java
public class UserRequest extends Message {

    static final long serialVersionUID = 20050617;
    public static final String MSGTYPE = "BE";
    

    public UserRequest() {
        super();
        getHeader().setField(new quickfix.field.MsgType(MSGTYPE));
    }
    
    public UserRequest(quickfix.field.UserRequestID userRequestID, quickfix.field.UserRequestType userRequestType, quickfix.field.Username username) {
        this();
        setField(userRequestID);
        setField(userRequestType);
        setField(username);
    }
    
    public void set(quickfix.field.UserRequestID value) {
        setField(value);
    }

    public quickfix.field.UserRequestID get(quickfix.field.UserRequestID value) throws FieldNotFound {
        getField(value);
        return value;
    }

    public quickfix.field.UserRequestID getUserRequestID() throws FieldNotFound {
        return get(new quickfix.field.UserRequestID());
    }

    public boolean isSet(quickfix.field.UserRequestID field) {
        return isSetField(field);
    }

    public boolean isSetUserRequestID() {
        return isSetField(923);
    }

    public void set(quickfix.field.UserRequestType value) {
        setField(value);
    }

    public quickfix.field.UserRequestType get(quickfix.field.UserRequestType value) throws FieldNotFound {
        getField(value);
        return value;
    }

    public quickfix.field.UserRequestType getUserRequestType() throws FieldNotFound {
        return get(new quickfix.field.UserRequestType());
    }

    public boolean isSet(quickfix.field.UserRequestType field) {
        return isSetField(field);
    }

    public boolean isSetUserRequestType() {
        return isSetField(924);
    }

    public void set(quickfix.field.Username value) {
        setField(value);
    }

    public quickfix.field.Username get(quickfix.field.Username value) throws FieldNotFound {
        getField(value);
        return value;
    }

    public quickfix.field.Username getUsername() throws FieldNotFound {
        return get(new quickfix.field.Username());
    }

    public boolean isSet(quickfix.field.Username field) {
        return isSetField(field);
    }

    public boolean isSetUsername() {
        return isSetField(553);
    }

    public void set(quickfix.field.Password value) {
        setField(value);
    }

    public quickfix.field.Password get(quickfix.field.Password value) throws FieldNotFound {
        getField(value);
        return value;
    }

    public quickfix.field.Password getPassword() throws FieldNotFound {
        return get(new quickfix.field.Password());
    }

    public boolean isSet(quickfix.field.Password field) {
        return isSetField(field);
    }

    public boolean isSetPassword() {
        return isSetField(554);
    }

    public void set(quickfix.field.NewPassword value) {
        setField(value);
    }

    public quickfix.field.NewPassword get(quickfix.field.NewPassword value) throws FieldNotFound {
        getField(value);
        return value;
    }

    public quickfix.field.NewPassword getNewPassword() throws FieldNotFound {
        return get(new quickfix.field.NewPassword());
    }

    public boolean isSet(quickfix.field.NewPassword field) {
        return isSetField(field);
    }

    public boolean isSetNewPassword() {
        return isSetField(925);
    }

    public void set(quickfix.field.RawDataLength value) {
        setField(value);
    }

    public quickfix.field.RawDataLength get(quickfix.field.RawDataLength value) throws FieldNotFound {
        getField(value);
        return value;
    }

    public quickfix.field.RawDataLength getRawDataLength() throws FieldNotFound {
        return get(new quickfix.field.RawDataLength());
    }

    public boolean isSet(quickfix.field.RawDataLength field) {
        return isSetField(field);
    }

    public boolean isSetRawDataLength() {
        return isSetField(95);
    }

    public void set(quickfix.field.RawData value) {
        setField(value);
    }

    public quickfix.field.RawData get(quickfix.field.RawData value) throws FieldNotFound {
        getField(value);
        return value;
    }

    public quickfix.field.RawData getRawData() throws FieldNotFound {
        return get(new quickfix.field.RawData());
    }

    public boolean isSet(quickfix.field.RawData field) {
        return isSetField(field);
    }

    public boolean isSetRawData() {
        return isSetField(96);
    }
}
```

### Creating a MarketDataRequest message -  QuickFixJ vs the equivalent FsFIX code

QuickFIXJ

```Java
    MDReqID reqId = new MDReqID("MDRQ-" + String.valueOf(System.currentTimeMillis()));
    MarketDepth depthType = new MarketDepth(1);
    MDUpdateType updateType = new MDUpdateType(MDUpdateType.INCREMENTAL_REFRESH);
    MarketDataRequest mdr = new MarketDataRequest(reqId, subscriptionType, depthType);
    mdr.setField(updateType);
    MarketDataRequest.NoRelatedSym instruments = new MarketDataRequest.NoRelatedSym();
    instruments.set(new Symbol(currencyPair));
    mdr.addGroup(instruments);
    mdr.setField(new NoMDEntryTypes(2));
    MarketDataRequest.NoMDEntryTypes group = new MarketDataRequest.NoMDEntryTypes();
    group.set(new MDEntryType(MDEntryType.BID));
    group.set(new MDEntryType(MDEntryType.OFFER));
    mdr.addGroup(group);
```

FsFIX

```F#    
    let ms = System.DateTimeOffset.Now.ToUnixTimeMilliseconds()
    let mdr = MkMarketDataRequest(
                MDReqID (sprintf "MDRQ-%d" ms),
                SubscriptionRequestType.SnapshotPlusUpdates,
                MarketDepth 1,
                [MDEntryType.Bid; MDEntryType.Offer] |> List.map MkNoMDEntryTypesGrp,
                [MkInstrument (Fix44.Fields.Symbol "EUR/USD")] |> List.map MkMarketDataRequestNoRelatedSymGrp )
```




### Comparing the overall size difference of FIX4.4 generated code in FsFIX, QuickFixJ and QuickFixN

A simple measure of conciseness, that counts the number of non-whitespace, non-line terminator characters in FIX4.4 generated code for FsFIX, QuickfixJ and QuickfixN, for roughly equivalent functionality.
            
FsFIX:      1,688,059

QuickfixJ:  8,045,893

QuickfixN:  7,523,220

The numbers speak for themselves.

## Summary

Comparison with QuickFixN and QuickFixJ shows that FsFIX creates more reliable, concise code than other implementations. The gains are attributable to FsFIX's use of F# ADTs. 
