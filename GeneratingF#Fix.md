# Generating F# from FIX spec XML (and generating FIX xml from F#)



F# requires modules, types, function definitions etc to be defined before they are used. FIX has groups containing components and components containing groups, and so FsFIX.CodeGen needs to generate components and groups in such a way that the dependencies appear before the types that depend on them. Components and Groups are generated into a single module, called Fix44.CompoundItems for want of a better name. They have been sorted [topologically](https://en.wikipedia.org/wiki/Topological_sorting) to get the dependency order such that the code will compile.

FsFIX represents FIX Components are represented as F# ADTs, but they have no effect in the byte representation of the FIX message. 

Groups in the FIX4.4.xml spec are defined inline in their parent messages, groups (groups can contain groups) and components. There are many groups with the same name, sometimes containing the same members, sometimes not. FsFIX code generation finds the most common case of each set of groups with the same name, and gives it a separate F# definition with that name, other instances of groups types are generated with a name that has been prefixed with the of the containing type. 

### the most common 'Legs' group

```F#
type NoLegsGrp = {
    InstrumentLegFG: InstrumentLegFG // component
    }

### Another 'Legs' group containing extra fields, and with the name of the parent type 'TradeCaptureReport' prefixed to get the full type name.

```F#
type TradeCaptureReportAckNoLegsGrp = {
    InstrumentLegFG: InstrumentLegFG // component
    LegQty: LegQty option
    LegSwapType: LegSwapType option
    NoLegStipulationsGrp: NoLegStipulationsGrp list option // group
    LegPositionEffect: LegPositionEffect option
    LegCoveredOrUncovered: LegCoveredOrUncovered option
    NoNestedPartyIDsGrp: NoNestedPartyIDsGrp list option // group
    LegRefID: LegRefID option
    LegPrice: LegPrice option
    LegSettlType: LegSettlType option
    LegSettlDate: LegSettlDate option
    LegLastPx: LegLastPx option
    }
```
The 'base' group name in FsFIX F# is that of the field containing the number of repeated group instances suffixed with 'Grp'


## other FsFIX characteristics and features
- Multicase FIX fields are represented by Multicase ADT discriminated unions (DUs)
- Single value Fix fields are represented by a single-case DU, not raw strings, ints etc
- Fields do not know their tag, field read/write functions do know their tag
- Optional fields, groups or components are wrapped in the F# Option type
- FIX message reading detects unexpected fields in FIX buffers
- Field reading copes with unordered msg body fields (unordered compared the order in the fix spec) outside of groups
- Time and DateTime types cope with leap-seconds as per the FIX spec
- Group reading expects fields and sub-groups within the group to be ordered and sequential, but the group can start in any position
- Extra group instances are detected, if the number of group instances is stated to be 4 but 5 are present then this will be detected
- Unexpected fields for a given msg type are detected
- Fields which may contain field or tag-value separators, such as RawData, are handled correctly




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

