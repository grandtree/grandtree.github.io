Table of Contents

{{TOC}}

This note is mainly based on the docs below    

+ [MARKDOWN GUIDE of iA Writer](https://ia.net/writer/support/general/markdown-guide/)

then it is copied to be variant for the syntax used by Marked 2, I will make
change in this doc once I find diff between Marked 2 and iAWriter.
In this note, for most examples of marks, I only write down the markdown code in
original text, final formatted text should be seen with Marked 2. 


# Basic formatting
## Line break
When you do want to insert a \<br/\> break tag using Markdown, you end a line with two or more spaces, then type return. Just like this:   
This is a new line   
This is another new line
But this is **NOT** a new line  

A blank line in original text file also has 

one corresponding blank line in final file rendered. 

My understanding is that here a blank line starts a new paragraph, just like it
does in TeX.   
iA Writer also allows you to insert a \<br/\> into a paragraph by using Shift+Enter. This will add the spaces required at the end of the line for you.  
Some Markdown editor will automatically add syntax mark for the enter key, for example, if I hit "Enter" key at the end of one list item to start a new line (but not a new list item), in this case, the "*" is automatically added, I need to use "Delete" key to remove "*" for just a new line, an alternative way is to use "Shift" and "Enter" at the same time. 

Some important notes   

+ Some elements have builtin line break, such as the headings. 
+ Most marks described in this note need one blank line at the beginning and the end to take effective. 

## Characteristics of font
Italics   
*example* or _example_   
Bold   
**example** or __example__   
Italics and bold   
***example*** or ___example___

Note that there should be no space between the * and words to be characterized.

## List
**Numbered list**   
Type 1. then a space. Any number (followed by a full stop and space) can be used and the list items will be ordered from 1 when exported.   

1. The first ordered item
2. The second ordered item
2. The third ordered item

Note that I don't need to worry about the value of numbers in original text, they don't matter for the formatted text. 

**Bulleted list**   
Type *, - or + then a space.

+ Bulleted list item 1
+ Bulleted list item 2

Let me start a new bulleted list with *

* Bulleted list item 1
* Bulleted list item 2

Let me start a new bulleted list with -

- Bulleted list item 1
- Bulleted list item 2

We can see that *, - and + all lead to the same bullet in formatted text. 

**Line break inside/outside list**   
One line break is automatically added for one list item, but inside the list item, line break should be added explicitly if needed.    
List needs one blank **before/after** the first/last item, I think it is count-intuitive, but more Google search shows that it seems like it is by design (for better readability).


**Task list**   
Type -[ ] or 1.[ ] then a space. Adding an x between the square brackets will tick off a task list item in the Preview.<br/>

- [ ] Unfinished task list item
- [x] Finished task list item   


**Nesting lists**   
Nested unordered list   

* First level
** Second level

Nested ordered list

1. First level
1.1. Second level
1. Second item in the first level
2. Third item in the first level



Nested mixed list

* First level unordered list item
* 1. Second level ordered list item
* 1. * Third level unordered list item
* 2. Second level ordered list item   
Note that at least this item is not formatted in the preview of iA Writer, I should be cautious to use nested mixed list. 

## <a name="table"></a>Table
To make a table, use vertical bar characters to denote cells. Start with column headers, separate with a row of cells with hyphens, then add further rows of cells. For example:

|Header |Column 1 | Column 2 | Column 3  | 
|:--- |:---- |:----:| ----:|
|1. Row| is | is | is  |
|2. Row| left | nicely | right  |
|3. Row| aligned | centered | aligned  |   

Pay attention to how ":" is used to specify the horizontal alignment. 

## Heading
Markdown supports up to six levels by writing # at the start of a line; the number of hashtags defines the hierarchy of the heading.   
Refer to the original text of headings in this note for examples. 
## Code
You can mark up code in-line using backticks as below   
`code line 1
code line 2`    
we can see that a explicit line break after ` is needed compared to the other ways below,     
or add a code block by adding at least four spaces to the start of a line:

    This is a code block
    This is another code line

This way is better for big code block, but note that it requires one blank line **before** it in the source of markdown file, or it will NOT be recognized as code block, the blank line after it is not needed, if used, it will be a blank line in the rendered markdown file.   
In addition, you can use Fenced code blocks, which begin and end with triple backticks, and don’t need indenting. Note that inline formatting (like _underscores_) is ignored in code.
```
This is a fenced code block
```

Don't use code block inside list item. 

## Blockquotes
Type > plus a space (just like email):
> A quoted paragraph
>> A quoted paragraph inside a quotation

## Links
Create a link by surrounding the link text in square brackets, followed immediately by the URL in parentheses:   
[text to link](http://example.com/)

You can also use reference links. Add the reference in square brackets after the text to link. Then, on a line by itself add the reference with a colon, space, and the URL:

I get 10 times more traffic from [Google] [1] than from
[Yahoo] [2] or [MSN] [3]. So [Google] [1] are popular.

  [1]: https://google.com/
  [2]: http://search.yahoo.com/
  [3]: https://search.msn.com/
You see that reference links can save typing if one link is referred multiple times. 

### Named anchor
This is [one internal link to the table section](#table).
## Images
Both local and web URLs are supported. On export to .docx, currently the image’s alt text is added, but the image is not embedded. Markdown uses the following syntax for images:

![alt text](https://daringfireball.net/graphics/logos/)
![Alt text](/path/to/img.jpg "Optional title")
## Horizontal rules
You can add a thematic break which will be represented by a dividing line (\<hr\>) when exported to HTML. To do so, add three or more asterisks (*), hyphens (-), or underscores (_) on a line by themselves, optionally separated with spaces. For example:

* * *
or

-------------

# Advanced formatting
## Footnotes
Add a footnote in square brackets preceded by a caret. Then add the footnote content like a reference link, for example:
Some text with a footnote[^1].

…

[^1]: The linked footnote appears at the end of the document.

You can also add an inline footnote in the following manner:
Some text with a footnote[^This is the footnote itself.].
## “Escaping” formatting characters  
If you want to type a formatting character and have Writer treat it as text not formatting, type a backslash first \. This means \\\* gives \*, \\\_ gives \_ etc. Escaping isn’t needed in code blocks.
## Meta Data
Use MultiMarkdown’s metadata to store important information about your documents, hidden from Preview. To start using metadata in your documents, turn on “Process Metadata” in Preferences.

You can do this with two easy steps. First you define your Metadata at the very top of your document, followed by an empty line. Let’s write “The Cat sat on the Mat” with Metadata. Write:   
Animal: Cat  
Thing: Mat  
You can use the metadata in the text by putting it in brackets adding a % sign. Write:   
The [%Animal] sat on the [%Thing].   
The whole document should now look like this:   
Animal: Cat  
Thing: Mat

The [%Animal] sat on the [%Thing].

If you open Preview and compare the raw text and the rendered Markdown you will see this:   
The Cat sat on the Mat





