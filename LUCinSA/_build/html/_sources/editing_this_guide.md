# To edit this guide
================================================================================================================================
```{figure} /Images/logo.png
:scale: 50%
:name: logo.png
Note this is a Jupyter Book. 
```
For more information, see the [Jupyter Book guide](https://jupyterbook.org/intro.html).

This is written in myST Markdown. For formatting, see the [myST Syntax guide](https://myst-parser.readthedocs.io/en/latest/syntax/syntax.html). Jupyter Book allows us to get fancier by writing directives. See the[MyST documentation](https://myst-parser.readthedocs.io/)

For information on publishing (for future), see [this guide](https://github.com/pabloinsente/jupyter-book-tutorial)


## Building the book

After any edits, the book needs to be rebuilt: 

```
#First activate virtual environment:
####(If using Git Bash, need to activate conda first with "conda activate")
####(If Git Bash does not recognize conda yet, see link below)
conda activate LUCenvWin
#Then build book (from inside LUCinSA directory):
jupyter-book build guide/
```
[see this guide to use conda in Git Hub](https://discuss.codecademy.com/t/setting-up-conda-in-git-bash/534473)

:::{admonition}Note if using Windows:
**Jupyter Book is not compatible with Python 3.8** in Windows. Set up environment using Python 3.7.
A Conda Environment .yml (LUCenvWin) is provided with settings for Windows. After creating and activating virtual environment,
need to install Jupyter Book: pip install jupyter-book
:::
