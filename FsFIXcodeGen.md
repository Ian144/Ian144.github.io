# FsFIX Code Generation

## Generating Compilable F# From The XML FIX specification





different fix4.4.xml's
    c# and java

FsFIXGen 
By default FsFIXGen loads a copy of the QuickFixN version


any small change to fix4.4.xml required
merging of groups
eliding the group-length field
    the length field is written and read, the value being derived from the list of group instances
merging of len+data fields
    the length field is written and read, the value being derived from the length of the data
topological sorting
    for F# dependency
rules
code-size comparison, between FsFIX and QuickFixN







- merging FIX length+data field pairs
- topological sorting of groups and components 
- merging groups with the same structure. FIX groups are defined in the messages or components which contain them, where possible FsFIX merges common group definitions into a single ADT group definition.
- the most common case is merged
  other possible merges are left with the name of their parent
  topographical sort of groups and components so they are generated in dependency order, if groupX has a componentY member then the componentY ADT needs to be defined in source before group
OTHER RULES (take from F# code) HERE

## FsFIX features
## FsFIX characteristics
- FIX Components are represented as ADTs so work with intellisense, but have no effect in the byte stream representation of the FIX message. 
- Multi-case FIX fields are represented by Multi-case ADT discriminated unions (DUs)
- Single value Fix fields are represented by a single-case DU, not raw strings, ints etc
- Fields do not know their tag, field read/write functions do know their tag
- Optional fields, groups or components are wrapped in the F# Option type
- FIX message reading detects unexpected fields in FIX buffers
- Field reading copes with unordered message body fields (unordered compared the order in the FIX spec)
- Time and DateTime types cope with leap-seconds as per the FIX spec
- Group reading expects fields and sub-groups within the group to be ordered and sequential, but the group can start in any position
- Extra group instances are detected, if the number of groups is stated to be 4 but 5 are present then this will be detected
- Unexpected fields for a given message type are detected
- Fields which may contain field or tag-value separators, such as RawData


## Code size FsFIX vs QuickfixN
    NOTE TO SELF - THIS MAY NOT BE BEING FAIR TO QUICKFIX, as FsFIX WILL REQUIRE Make<MSGNAME> TO HAVE THE SAME CONVENIENCE WHICH MEANS ADDING 

    quickfixN message definitions: 14.4MB in 94 files
    quickfixN message definitions: 
    quickfixN field definitions: 647kb non comment bytes in 1 file (seems to be a single file for all FIX versions, 999 fields)

    Fix44.Messages.fs: 87.2kb
    FsFix field definitions FIX4.4 only: 117kb in 1 file (for FIX4.4 only 900 fields, 916 before len+data merge )


