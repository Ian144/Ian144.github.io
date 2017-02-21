# FsFIX Code Generation


## Generating FsFIX F# from FIX4.4.xml and generating FIX4.4.xml from FsFIX

### TBD




## FsFIX features and characteristics

- Multi-case FIX fields are represented by Multi-case ADT discriminated unions (DUs)
- Single case fields are wrapped in a single-case DU
- Fields do not know their tag, field read/write functions do know their tag
- Optional fields, groups or components are wrapped in the F# Option type
- Field reading copes with unordered message body fields (unordered compared with the order in the FIX spec)
- Unexpected fields for a given message type are detected
- Fields which may contain field or tag-value separators, such as RawData, are handled correctly
- Time and DateTime types cope with leap-seconds as per the FIX spec

- FIX Components are represented as ADTs in F#, and so work with intellisense, but have no effect in the byte representation of the FIX message. 

- Group reading expects fields and sub-groups within the group to be ordered and sequential
- Extra group instances, in a byte buffer containing a message, are detected.







### FsFIX merges length + data field pairs

FIX specifies some related pairs of fields, whereby one field contains data and the preceding field contains the length of the data. The data field may contain FIX field and tag-value separators, though these must not be treated as seperators. Quickfix defines both fields separately, FsFix elides the length field.
```Java
public class RawDataLength extends IntField {
    static final long serialVersionUID = 20050617;
    public static final int FIELD = 95;

    public RawDataLength() {
        super(95);
    }

    public RawDataLength(int data){
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


### FsFIX does not store the number of repeating groups

FIX repeating groups are associated with a field containing the number of repeats. FsFIX generated code stores the group instances in a list, the number of repeating groups is the length of the list. The 'number of groups' field is read and written by 











