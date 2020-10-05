# endnote_convert
Collection of scripts to collect plain text references and convert them back into editable citations by endnote.

I don't care about the details, [jump to the workflow](#How-To)

## Idea and concept
Sometimes you have an old manuscript draft, where the references have been converted into plain text and can't be changed by [Endnote](https://endnote.com/) anymore. There is no inbuild option to reverse this and you are stuck with either updating everything by hand or starting over by inserting all the references anew.
This workflow will hopefully help you with this and covert your plain text references into a list of endnotes that Endnote can modify.
The basic concept is that the references will be imported into Endnote as a **new** library, because Endnote numbers all entries in the order that they are created. This means if we have references in the form of <sup>1, 2, 3</sup> we will import them in exaclty this order. Later on, the Microsoft Word file will be modified to exchange all occucences of <sup>1, 2, 3</sup> into {#1}{#2}{#3}, tricking Endnote into thinking these are unformatted citations and that the numbers correspond with the numbers in the library. 

## Requirements
Written in Python 3.8.5 using to following modules:
* sys
* os.path
* re
* string
* requests
* random
* BeautifulSoup
* quote

## The scripts
**scrape_refs.py**
So far, works only with [Pubmed](https://pubmed.ncbi.nlm.nih.gov/) for biomedical science. I tried google scholar, but could not figure out how to make it work, so I went for my field of expertise.
The script takes a list of references from a file and scrapes Pubmed for each reference. The references need to be provided in the form of **one reference per line**, e.g.
* Author et al, Amazing fancy title. YourFavJournal (2020)
* Foo, B. and Bar, F.. Plants make oxygen, maybe? Journal of House plants. (1990)
* etc.

It also works with numbered lists like
1. Author et al, Amazing fancy title. YourFavJournal (2020)
1. Foo, B. and Bar, F.. Plants make oxygen, maybe? Journal of House plants. (1990)
1. etc.

Please be aware that in this case the numbers will be part of the search string. Usually, this does not matter, but if you encounter problems try to remove them.
Empty lines are skipped.
The output will be written in a file called *references.txt* in the National Library of Medicine format, which can be read by essentially all reference managers.

This means that this script **can be used by anyone to extract references from a list into your favorite reference manager** in a semi-automatic way.

**endnote_convert.py**
This script simply replaces all occucences of <sup>1, 2, 3</sup> into {#1}{#2}{#3}.
However, working with docx files in Python is not very intuitive. For this reason, the ability of word to save and open websites or simply HTM(L) files is used.
Word converts all superscript citiations into this code
```
<sup><span style="mso-no-proof:yes">1, 2, 3</span></sup>
```
**However, this code snippet was generated by Word 2011 and might be dependent on the version of Word that is used.** I was so far unable to test different versions of Word and how they handle the conversion into html.
After running the python script over the html file, it can be opened in Word again and all citiation should be converted into unformatted citations.

## How To
1. Make sure your python is up to date and you have all the [required modules](#Requirements) installed.
1. Download both python scripts and save them somewhere, for example on your desktop inside an extra folder.
1. Copy and paste the citation list from your plain text Word file into a new text file. To get rid of all formatting, use something simple like notepad (win), textedit (mac), or sublime. **One reference per line.** Save it under a simple name you like ("refs.txt") in the same folder as the python scripts to keep things simple.
1. Make a copy of your Word file in the same folder as everything else.
1. Open this new copy of the Word file and navigate to "Save as...". Select Website, Web Page or HTML. **Do not save yet!** Please check the encoding of the file, which can be found under Options or Web Options in the "Save as..." window. There should be a tab called "encoding", select **UTF-8** if not already selected. Save the HTML document into the same folder as everything else.
1. Open Endnote and create a new, empty library, the name does not matter. Maybe use the same folder as for the other files again?
1. Open a console/terminal window and navigate to the folder that contains all the files from the previous steps.
1. Start *scrape_refs.py*, it will ask for the name of the file that contains the citations. It will start scraping Pubmed for the references, this will take a while, dependent on the number of references. The output in the console/terminal window will tell you what is currently happening. If a citation is not found, this will be noted and listed at the end.
1. If all citations have been found, proceed with the next step. If not, try to find out why. Try to modify the entry in your reference file, e.g. remove any unnecessary numbers or long and complicated Journal names. Swap the year around or add/remove braces. **You can run the script as often as you like, the output file *bibliography.txt* will be replaced every time.**
1. Open the new and empty Endnote library. Go to "Import..." and select "Pubmed (NML)" as filter. Now select your *bibliography.txt*. **Check that the correct number of citations is imported.** Sometimes Endnote skips a citation for some unknown reason. I found that it helps to open *bibliography.txt* with a simple text editor (like notepad, textedit or sublime) and just save the file again. Repeat the import.
1. Use the same console/terminal window and run *endnote_convert.py*. It will ask again for a name, this time of the HTML file you saved from Word. You should just get a message that the references were exchanged, which sould be fairly quick. The new file will be save under the name *Yourfile_modified.htm*.
1. Open the modified HTML file in Word, you should be able to perform right-click -> open with Word.
1. Go to the Endnote tab und look for 'Configure Bibliogrphy" and check if the "Temporary citation delimiters" are set to { and } (they should be). Press OK, Endnote should now start to search for unformatted citations and replace them. If it doesn't, look for "Update Citations and Bibliography" in the Endnote tab.
1. Save your Word file as .docx. You are done and should be able to edit your references now.

## Known limitations
* The script only works with Pubmed so far. If you can make google scholar work, feel free to fork the project.
* Sometimes Endnote does not properly import all citations from the *bibliography.txt* file. I suspect that it has something to do with encoding, but I'm unable to pinpoint the problem. Simply opening in a text editor like subline and saving the file again got rid of this problem for me.
* I don't know how other Word versions perform the conversion into HTML. If another structure of the superscpt citations is used, exchangig the regular expression in the file will probably help.
* *endnote_convert.py* does not work with citation styles in the form of (Author et al. 2020), only <sup>1, 2, 3,</sup> is currently supported. I might work on this in the future, but no promises. [There seems to be a workaround](https://community.endnote.com/t5/EndNote-How-To/Converting-manually-created-in-text-citations-to-EndNote/m-p/135008/highlight/true#M25047) and *scrape_refs.py* can still be used to generate a new Endnote library.

## References
Idea for the workflow: https://community.endnote.com/t5/EndNote-How-To/Converting-manually-created-in-text-citations-to-EndNote/td-p/135007

Initial script idea for *scrape_refs.py*: https://blog.macuyiko.com/post/2016/converting-plain-text-references-to-bibtex-updated.html
