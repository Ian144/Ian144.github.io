## THIS ARTICLE IS WORK-IN-PROGRESS


# The Financial Information eXchange (FIX) Protocol in F# #


There is a myth that F# is only useful for mathematical and scientific applications. This article describes using F# to model [FIX](https://en.wikipedia.org/wiki/Financial_Information_eXchange) messages, which is neither mathematical or scientific. Why should you care? what does F# give you that C# or Java does not? among other things, FsFIX is going to show that with F# some runtime errors become compile time errors, that property based testing (possible in but not facilitated by C# and Java) is supercharged compared to unit testing<sup>1</sup> and will catch some of the runtime errors that remain, and that the code base is smaller and has less boilerplate.

FIX message code generation is the first stage of project to build a fully fledged FIX engine using F#. I have not heard of anyone using a functional programming language to build a FIX engine, I thought ADTs would be a good match for FIX messages and that it would be interesting to try. For performance reasons a more imperative coding style has been used for the message reading and writing functions, though this code has plenty of scope for being tuned.

The source for FsFIX can be found [here](https://github.com/Ian144/fsFixGen). At the moment FsFIX can be used to connect to applications using other FIX engines and feed them with random but valid FIX messages, the idea being to test the parsing machinery of both FIX frameworks. Errors were found in QuickFIXN (an open-source C# FIX engine) and FsFIX (found and fixed) in this manner. 

[The unreasonable effectiveness of Algebraic Data Types](ADTs.md)

[Generating F# from FIX4.4.xml](FsFIXcodeGen.md)
- should this be a full sub-article??
- should this include "FsFIX merges length + data field pairs"
- Code size comparison between FsFIX and QuickFixN

[Property testing FsFIX message read-write code](PropertyTesting.md)

[Performance](Performance.md)

[Using FsFIXEcho to exercise QuickFixN and QuickFixJ](FsFIXEcho.md)


## my other site, an F# key-value store: https://fredisnet.org/

## interested in hiring me, ? my email address is my GitHub username followed by '@hotmail.com'


1. unit tests are still useful, property testing is not always appropriate.





