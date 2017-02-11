## THIS ARTICLE IS WORK-IN-PROGRESS


# The Unreasonable Effectiveness of Algebraic Data Types - Representing the Financial Information eXchange Protocol (FIX) in F# #

Structs and classes are pretty much all there is when it comes to designing your own types in object oriented programming languages. The ML branch of functional programming, which includes F#, has an alternative - Algebraic Data Types (ADTs). A good introduction to ADTs, why they are useful, and using them in F# is Scott Wlaschins NDC talk: [Domain modelling with the F# type system](https://vimeo.com/97507575). ADTs can be concise, facilitate making [making invalid states unrepresentable](http://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/) and work well with property based testing, see [this](http://fsharpforfunandprofit.com/posts/property-based-testing) or [this](https://fscheck.github.io/FsCheck/QuickStart.html)

There is a myth that F# is only useful for mathematical and scientific applications. This article describes using F# ADTs to model [FIX](https://en.wikipedia.org/wiki/Financial_Information_eXchange) messages, this is neither a mathematical nor scientific application. FsFixGen F# projects can be found at [fsFixGen](https://github.com/Ian144/fsFixGen). FsFixGen creates ADTs to represent FIX messages, groups and fields from the same xml FIX specification used as source by the Java and C# versions of quickfix. FsFixGen also generates code to read and write FIX messages to and from byte arrays. The generated F# FIX code has been checked in, and can be found in the fsFix subproject. There are several different versions of FIX, FsFIXGen currently works for FIX 4.4, but should work with other versions with a little modification. This code generation is the first stage of project to build a fully fledged FIX engine using F#. I have not heard of anyone using a functional programming language to build a FIX engine, I thought ADTs would be a good match for FIX messages and that it would be interesting to try. I have avoided not writing or generating OO code in F#, C# is perfectly good for that, and I want to illustrate the advantages of the functional approach.


### Classes vs ADTs

Some FIX fields have a finite number of valid values, such as 'PosType', in quickFix Java this looks like


#### The PosType field in Java

```Java
public class PosType extends StringField {

    static final long serialVersionUID = 20050617;

    public static final int FIELD = 703;
    public static final String TRANSACTION_QUANTITY = "TQ";
    public static final String INTRA_SPREAD_QTY = "IAS";
    public static final String INTER_SPREAD_QTY = "IES";
    public static final String END_OF_DAY_QTY = "FIN";
    public static final String START_OF_DAY_QTY = "SOD";
    public static final String OPTION_EXERCISE_QTY = "EX";
    public static final String OPTION_ASSIGNMENT = "AS";
    public static final String TRANSACTION_FROM_EXERCISE = "TX";
    public static final String TRANSACTION_FROM_ASSIGNMENT = "TA";
    public static final String PIT_TRADE_QTY = "PIT";
    public static final String TRANSFER_TRADE_QTY = "TRF";
    public static final String ELECTRONIC_TRADE_QTY = "ETR";
    public static final String ALLOCATION_TRADE_QTY = "ALC";
    public static final String ADJUSTMENT_QTY = "PA";
    public static final String AS_OF_TRADE_QTY = "ASF";
    public static final String DELIVERY_QTY = "DLV";
    public static final String TOTAL_TRANSACTION_QTY = "TOT";
    public static final String CROSS_MARGIN_QTY = "XM";
    public static final String INTEGRAL_SPLIT = "SPL";
    public static final String RECEIVE_QUANTITY = "RCV";
    public static final String CORPORATE_ACTION_ADJUSTMENT = "CAA";
    public static final String DELIVERY_NOTICE_QTY = "DN";
    public static final String EXCHANGE_FOR_PHYSICAL_QTY = "EP";
    
    public PosType() {
        super(703);
    }

    public PosType(String data) {
        super(703, data);
    }
```

The Java PosType class stores the case as a string, there are string constants defined, but their use is not checked. Java is a statically typed language, but it is not taking advantage of type checking, the code below compiles.

```Java
quickfix.field.PosType pt = new quickfix.field.PosType("any old string");
```

#### The PosType field represented as an F# ADT #
```fsharp
type PosType =
    | TransactionQuantity
    | IntraSpreadQty
    | InterSpreadQty
    | EndOfDayQty
    | StartOfDayQty
    | OptionExerciseQty
    | OptionAssignment
    | TransactionFromExercise
    | TransactionFromAssignment
    | PitTradeQty
    | TransferTradeQty
    | ElectronicTradeQty
    | AllocationTradeQty
    | AdjustmentQty
    | AsOfTradeQty
    | DeliveryQty
    | TotalTransactionQty
    | CrossMarginQty
    | IntegralSplit
```
In F# it is impossible for PosType to have a case not defined in the type definition, an attempt to set an invalid state is a compile error.


### FIX length + data field pairs

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


```fsharp
type RawData =
    |RawData of NonEmptyByteArray.NonEmptyByteArray
     member x.Value = let (RawData v) = x in v
```






































## Codesize FsFIX vs quickfixN












[FsFixInner](FsFix Features)








You can use the [editor on GitHub](https://github.com/Ian144/Ian144.github.io/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url)
![Image](src)


For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/Ian144/Ian144.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
