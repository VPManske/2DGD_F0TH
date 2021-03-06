F.A.Q. and Guides
=================

I can't export my diagrams to Vector PDF, what do I do?
--------------------------------------------------------

If you can't export your diagrams in PDF, try exporting them in the `*.eps` format and then use pdflatex's `epspdf` tool, like follows:

`epspdf file_name.eps`

Some warnings may pop out, check the resulting pdf and if all is good, you can now add it to the book.

How do I know if my diagrams are in Vector PDF?
-----------------------------------------------

If your PDF files were created by converting from a `*.svg` or `*.eps` file, there is already a good chance that they're Vectors.

To check, just open the diagram with your favourite PDF reader and zoom into it: if the lines and words stay crisp (or are blurry for a moment and then go back to being crisp) you have a Vector PDF in your hands.

Adding a new programming language
---------------------------------

Latex (which is the engine that renders this book) supports a lot of programming languages already, with its `listings` package, but if your programming language is not there, or you want to translate the book to a specific framework, you can create a new language specification to use.

To do that, you can create a new `.tex` file in `template/additional_languages` and use the `\lstdefinelanguage` command to define the keywords to highlight. Check the `listings` package documentation for more information.

When you're done, just make a pull request and I will take care of integrating it into the book.

In case you want to integrate it yourself, just add an `\input` statement in the `template/template.tex` file.
