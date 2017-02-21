


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



