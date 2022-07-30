---
title: Restore raw latex file from existing pdf  
date: 2021-08-06 19:13:20
tags:
- Paper
categories: 
- Paper
---
Maybe you have lost the raw latex file due to some strange reasons. Unforturenatly, there is no some simple way to rapidly restore the raw Tex code from the compiled pdf(Portable Document Format) file. Don't be upset, Let's restore it as far as possible. 

## Text

First of all, the restoration of text from the pdf file is simple but tedious. Nevertheless, there are also some troubles, it needs to choose a strong pdf reader to avoid the problem that the copied text with format from some pdf reader may cause the complicated layout and unnecessary space. This blog opens the pdf file by Adobe Acrobat and saves it as the docx file, then you can copy the text without any format to the Tex file from the docx file.

## Image

Since the images are not embedded inside a pdf file but saved as raw binary code, it is not easy to extract raw images from an existing pdf file. If you need to get vector graphics, the simplest way is that extract vector graphics from the pdf file by some vector software, i.e., Affinity Designers, Inkscape, Adobe Illustrator. This blog chooses Affinity Designers(Paid Software) because it is convenient to choose vector graphics and extract them.

## Table

This blog chooses [tablesgenerator](tablesgenerator.com) to rapidly restore tables of lost paper. Firstly, we obtain the docx file as the Text section. Then we copy a table and save it as a CSV file. Next, we upload the CSV file to the website and adjust the format. Finally, you can copy the generated code to the Tex file.

## Algorithm

The restoration of Algorithm is similar to the Text Section. It is best to copy text from the generated docx file. Note that  algorithm2e package is easier to use than algorithm with algorithmicx.

## Equation

[Mathpix Snip](https://mathpix.com/) is a great OCR software to covert images of mathematical formulas to Tex code or other formats such as word. It is easy to accurately get raw Tex code of mathematical equations through the software.

## References

About references, we recommend that: 

1. Search all references through Google Scholar and save them(Click the star icon below the document item).
2. Open 'My library' and export all items in the BibTeX format in Google Scholar.
3. Then, you can use a BibTeX file to manage cited references or [convert them to Bibitem](https://tex.stackexchange.com/questions/124874/converting-to-bibitem-in-latex).

