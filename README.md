## The Unreasonable Effectiveness of Algebraic Data Types - Representing the Financial Information eXchange Protocol (FIX) in F# #

Structs and classes are pretty much all there is when it comes to designing your own types in object oriented programming languages. The ML branch of functional programming, which includes F#, has an alternative - Algebraic Data Types (ADTs). A good introduction to ADTs and using them in F# is [Scott Wlaschin - Domain modelling with the F# type system](https://vimeo.com/97507575). ADTs can be concise, facilitate making [making invalid states unrepresentable](http://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/) and  [property based testing](http://fsharpforfunandprofit.com/posts/property-based-testing)


This article describes using ADTs to model [FIX](https://en.wikipedia.org/wiki/Financial_Information_eXchange) messages, the GitHub repository is [fsFixGen](https://github.com/Ian144/fsFixGen). FsFixGen creates ADTs to represent FIX messages, groups and fields from the same xml FIX specification used as source by the Java and C# versions of quickfix. FsFixGen also generates code to read and write FIX messages to and from byte arrays. There are several different versions of FIX, FsFIXGen currently works for FIX 4.4, but would work with other versions with little modification.



[https://en.wikipedia.org/wiki/Algebraic_data_type](https://en.wikipedia.org/wiki/Algebraic_data_type)

there is a myth that F# and similar languages are best used for mathematical purposes
i hope this shows that the attributes of F# and functional programming are applicable outside of the mathematical domain
am not writing OO code in F#, C# is perfectly good for writing OO
github repo url

unlike enums it is impossible to have invalid instances


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

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/Ian144/Ian144.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
