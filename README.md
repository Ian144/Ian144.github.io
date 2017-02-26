## THIS ARTICLE IS WORK-IN-PROGRESS


# The Financial Information eXchange (FIX) Protocol in F# #


There is a myth that F# is only useful for mathematical and scientific applications. This site describes using F# to model [FIX](https://en.wikipedia.org/wiki/Financial_Information_eXchange) messages, which is neither mathematical or scientific. Why should you care? what does F# give you that C# or Java does not? FsFIX is going to show that 

1. with F# some runtime errors become compile time errors.

2. property based testing (possible in but not facilitated by C# and Java) is supercharged compared to unit testing<sup>1</sup> and will catch some of the runtime errors that remain.

3. that the generated code base is smaller and has less boilerplate.

 FsFIX generates F# types to represent FIX messages, and also generates functions to read and write these types to and from byte arrays. For performance reasons a more imperative coding style has been used for the message reading and writing, though this code has plenty of scope for being tuned. FIX code generation is the first stage of project to build a fully fledged FIX engine using F#.

The source for FsFIX can be found [here](https://github.com/Ian144/fsFix). At the moment FsFIX can be used to connect to applications using other FIX 4.4 engines and feed them with random but valid FIX messages, the idea being to test the parsing machinery of both FIX frameworks. Errors were found in QuickFIXN (an open-source C# FIX engine) and FsFIX (found and fixed) in this manner, as was an issue with the FIX 4.4 spec

[The unreasonable effectiveness of Algebraic Data Types](ADTs.md)

[Generating F# from FIX4.4.xml (and generating FIX xml from F#)](GeneratingFsFix.md)

[Property testing FsFIX message read-write code](PropertyTesting.md)

[Performance](Performance.md)

[Building and Running FsFIX projects](BuildRun.md)



## About the author

My other site, an F# key-value store: https://fredisnet.org/

Interested in hiring me or asking me questions about FsFIX? my email address is my GitHub user-name followed by '@hotmail.com'



<sup>1</sup> unit tests are still useful, property testing is not always appropriate.
