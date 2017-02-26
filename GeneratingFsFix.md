# Generating F# from FIX spec XML (and generating FIX xml from F#)


## Generating components and group F# types in dependency order

F# requires modules, types, function definitions etc to be defined before they are used. FIX has groups containing components and components containing groups, and so FsFIX.CodeGen needs to generate components and groups in such a way that the dependencies appear before the types that depend on them. Components and Groups are generated into a single module, called Fix44.CompoundItems for want of a better name. FsFIX component and group definition order has been sorted [topologically](https://en.wikipedia.org/wiki/Topological_sorting) so that the code will compile.

FIX Components are represented in FsFIX, but they have no effect in the byte representation of the FIX message. 


## Merging repeating groups

Groups in the FIX4.4.xml spec are defined inline in their parent messages, groups (groups can contain groups) and components. There are many groups with the same name, sometimes containing the same members, sometimes not. FsFIX code generation finds the most common case of each set of groups with the same name, and gives it a separate F# definition with that name, while other instances of groups with the same base name are generated with a name prefixed with that of the containing type. 

#### The most common 'Legs' group

```F#
type NoLegsGrp = {
    InstrumentLegFG: InstrumentLegFG // component
    }
```

#### A 'Legs' group containing extra fields and groups

and with the name of the parent type 'TradeCaptureReport' prefixed to get the full type name.

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
(The 'base' group name in FsFIX F# is that of the field containing the number of repeated group instances suffixed with 'Grp'.)


## Merging length + data field pairs

FIX specifies some related pairs of fields, whereby one field contains data and the preceding field contains the length of the data. The data field may contain FIX field and tag-value separators, though these must not be treated as separators. Whereas QuickfixJ and QuickfixN define both fields separately, FsFix elides the length field. The length field is read from or written to the FIX buffer by the functions with read and write the data field, so that code using FsFIX does not have to.

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

### Simple string, int etc FIX fields are wrapped by a single-case discriminated union.

Once wrapped, type checking will raise compilation errors if code attempts to assign an AdvID value to an Account field

```F#
type Account =
    |Account of string
     member x.Value = let (Account v) = x in v


type AdvId =
    |AdvId of string
     member x.Value = let (AdvId v) = x in v
```


## FsFIX code generation rules

1. Ensure a group's first item is always required. If this was not done then groups are difficult if not impossible to parse. I suspect all FIX engines do this.

2. Ensure components that are the first item of a group have a first item that is required. As components are elided in the FIX message byte representation this is effectively an extension to rule '1'.

3. If an optional component contains just a single optional group then replace the component with the group. I think components containing just a single group definition is FIX4.4.xml's way of defining a group that can be used in multiple places. Group definitions are inline with their parents. The parent component is not required once the xml has been transformed into F#.

4. Promote 'optional' components to 'required' if the component contains only optional items. To have both the component and its sub-items as optional leads to ambiguous states with the same bit represention when sent down the wire e.g. the optional component is Option.Some when all the sub-items are Option.None VS the component is Option.None.
To remove this ambiguity while keeping components first class (for fsFix user convenience) in FsFIX such components are made 'required'. Optionality is preserved because the sub-items are optional.


## Other FsFIX characteristics and features

- Fields do not know their tag, whereas field read/write functions do know their tag.
- Optional fields, groups or components are wrapped in the F# Option type.
- FIX message reading detects unexpected fields in FIX buffers.
- FIX message reading copes with unordered fields (unordered compared to the order in the FIX4.4.xml spec) outside of groups.
- Unexpected fields for a given msg type are detected.
- Group reading expects fields, components and sub-groups within the group to be ordered, but the group can start in any position.
- Extra or missing group instances are detected when reading a FIX message buffer, if the number of group instances is stated to be 4 but 5 are present then this will be detected.
- Time and DateTime types cope with leap-seconds.


## Generating FIX4.4.xml from FsFIX F# #

The FsFIX.ReverseXMLGen project recreates the \<messages> and \<components> elements of FIX4.4.xml from FsFIX F# types. This is done as a check, to confirm that the merging of repeating groups has been done correctly, and to view the effect of the code generation rules mentioned above. This would be difficult with QuickFIX C# or Java code.

