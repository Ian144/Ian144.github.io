## THIS ARTICLE IS WORK-IN-PROGRESS


# Representing the Financial Information eXchange (FIX) Protocol in F# #


There is a myth that F# is only useful for mathematical and scientific applications. This article describes using F# to model [FIX](https://en.wikipedia.org/wiki/Financial_Information_eXchange) messages, neither a mathematical nor scientific application. Why should you care? what does F# give you that a C# or Java does not? among other things, this site is going to show that with F# some runtime errors become compile time errors, that property based testing (possible in but not facilited by C# and Java) will catch some of the runtime errors that remain, and that the code base is smaller and has less boilerplate.

FIX message code generation is the first stage of project to build a fully fledged FIX engine using F#. I have not heard of anyone using a functional programming language to build a FIX engine, I thought ADTs would be a good match for FIX messages and that it would be interesting to try. Writing or generating OO code in F# has been avoided, C# is perfectly good for that, I want to illustrate the advantages of the functional approach.  The source for this project can be found [here](https://github.com/Ian144/fsFixGen).


[The unreasonable effectiveness of Algebraic Data Types](ADTs.md)


[Property testing FsFIX message read-write code](PropertyTesting.md)

[Generating F# from FIX4.4.xml](xx.md)
- should this be a full sub-article??
- should this include "FsFIX merges length + data field pairs"
- Code size comparison between FsFIX and QuickFixN




## Hire Me, email ian144@hotmail.com to receive a CV








