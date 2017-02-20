

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


