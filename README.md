## THIS ARTICLE IS WORK-IN-PROGRESS


# The Financial Information eXchange (FIX) Protocol in F# #


There is a myth that F# is only useful for mathematical and scientific applications. This article describes using F# to model [FIX](https://en.wikipedia.org/wiki/Financial_Information_eXchange) messages, which is neither mathematical or scientific. Why should you care? what does F# give you that C# or Java does not? among other things FsFIX is going to show that with F# some runtime errors become compile time errors, that property based testing (possible in but not facilitated by C# and Java) is supercharged compared to unit testing<sup>1</sup> and will catch some of the runtime errors that remain, and that the code base is smaller and has less boilerplate.

FIX message code generation is the first stage of project to build a fully fledged FIX engine using F#. I have not heard of anyone using a functional programming language to build a FIX engine, I thought Algebraic Data Types (ADTs) would be a good match for representing FIX messages and that it would be interesting to try. For performance reasons a more imperative coding style has been used for the message reading and writing, though this code has plenty of scope for being tuned.

The source for FsFIX can be found [here](https://github.com/Ian144/fsFixGen). At the moment FsFIX can be used to connect to applications using other FIX 4.4 engines and feed them with random but valid FIX messages, the idea being to test the parsing machinery of both FIX frameworks. Errors were found in QuickFIXN (an open-source C# FIX engine) and FsFIX (found and fixed) in this manner, and also an issue with the FIX spec

[The unreasonable effectiveness of Algebraic Data Types](ADTs.md)

[Generating F# from FIX4.4.xml, and generating FIX xml from F#](FsFIXcodeGen.md)

[Property testing FsFIX message read-write code](PropertyTesting.md)

[Performance](Performance.md)


## FsFIX.sln projects
- Benchmark: times FsFIX reading and writing functions
- CodeGEN: generates F# FIX types from a FIX XML spec (currently only works with FIX4.4)
- FIXEcho: sends, receives and compares random but valid FIX messages sent to and received from some other FIX engine
- FsFIX: generated F# FIX type
- PropertyTests
- ReverseXMLGen: generates a (partial )FIX XML specification from F# FIX types
- UnitTests




## About the author

My other site, an F# key-value store: https://fredisnet.org/

Interested in hiring me or asking me questions about FsFIX? my email address is my GitHub user-name followed by '@hotmail.com'



<sup>1</sup> unit tests are still useful, property testing is not always appropriate.





