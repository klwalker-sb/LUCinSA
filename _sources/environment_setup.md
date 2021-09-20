---
jupyter:
  jupytext:
    text_representation:
      extension: .md
      format_name: pandoc
      format_version: 2.12
      jupytext_version: 1.11.4
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
  nbformat: 4
  nbformat_minor: 5
---

::: {.cell .markdown}
# \# Setting up your remote environment {\#-setting-up-your-remote-environment} {#-setting-up-your-remote-environment--setting-up-your-remote-environment}

## The general cluster environment

As with most cluster computers, the ERI system uses a UNIX-based shell called "BASH". This is a very simplified system that interprets code via command-line interface. To interact with the system, you need to use Shell scripting, which is based on UNIX/Linux command language. For a quick review of basic UNIX commands, [see here](Unix). For a more comprehensive coverage of working with BASH scripting, see these hpc-carpentry lessons on [navigating in bash](http://www.hpc-carpentry.org/hpc-shell/02-navigation/index.html) and [file manipulation in bash](http://www.hpc-carpentry.org/hpc-shell/03-files/index.html).

The point of high-performance computing is to manage and process large work loads. This is acheived through a work load manager, SLURM (Simple Linux Utility for Resource Management). Ultimate processing commands are sent to SLURM from .sh scripts in the bash directory via the command `sbatch`. Any other non-UNIX-based commands are processed within a virtual environment (e.g. via a Python interpreter).

## Setting up a virtual environment

Virtual environments allow us to customize our coding toolkit and preferences for each project within its own container so that it does not risk interfering with those of other projects or users. Within a virtual environment, you can define the version of Python that will be used, install packages specific to the project, and customize settings to facilitate your interactions, including aliases, which are shortcuts to replace frequently used or cumbersome commands. After connecting to bellows via ssh for the first time, you will need to set up your environment.
:::{admonition}Make sure you are in your home directory when creating/changing environment settings!
You should see your username \@bellows in the command prompt.
(there may be numbers in front; this is fine for now and can be taken care of later [here](profileIssue)).\
If you are unsure whether you are in your home directory, type cd \~/
:::

::: {.cell .markdown}
For this project, we will create a virtual environment in Python 3 called .nasaenv (output -\> \~/.nasaenv):

    python3 -m venv .nasaenv

***Activate your virtual environment:*** every time you run code that isn't BASH/SLURM from within the cluster:

    source .nasaenv/bin/activate

***Deactivate the virtual environment:*** when you are done:

    (.nasaenv) deactivate

### Install Python tools/packages and dependencies from project github scripts:

To **interact with github on your virtual computer**, you first need to feed the remote computer the password. You can do this with [an osxcechain credential](https://docs.github.com/en/get-started/getting-started-with-git/updating-credentials-from-the-macos-keychain) or by simply caching it by entering the following into your local git bash or mac terminal:
`git config --global credential.helper cache`
Or in git for Windows enter: `git config --global credential.helper wincred`

**While in your home directory,** enter the following commands to setup your Python environment:

    ​
    # Activate the virtual environment
    source .nasaenv/bin/activate
    ​
    # First update pip and other install tools
    (.nasaenv) pip install -U pip setuptools wheel
    ​
    # Install numpy and cython required for compiling .c code
    (.nasaenv) pip install cython numpy
    ​
    # Install extra dependencies not installed with packages below
    (.nasaenv) pip install gsutil descartes ray requests opencv-python
    ​
    # Install geowombatdev package from github(the dependencies here will take care of most other dependencies)
    (.nasaenv) mkdir tmp
    (.nasaenv) cd tmp/
    (.nasaenv) git clone https://github.com/jgrss/geowombatdev.git
    (.nasaenv) cd geowombatdev/
    (.nasaenv) python setup.py build && pip install .
    ​
    # Install eosvault package from github
    (.nasaenv) cd ~/tmp/
    (.nasaenv) git clone https://github.com/jgrss/eosvault.git
    (.nasaenv) cd eosvault/
    (.nasaenv) python setup.py build && pip install .
    ​
    # Install rastercrf from github
    (.nasaenv) cd ~/tmp/
    (.nasaenv) git clone https://github.com/jgrss/rastercrf.git
    (.nasaenv) cd rastercrf/
    (.nasaenv) python setup.py build && pip install .
    ​
    # Install satsmooth from github
    (.nasaenv) cd ~/tmp/
    (.nasaenv) git clone https://github.com/jgrss/satsmooth.git
    (.nasaenv) cd satsmooth/
    (.nasaenv) python setup.py build && pip install .
    ​
    # Install sacfei from github
    (.nasaenv) cd ~/tmp/
    (.nasaenv) git clone https://github.com/jgrss/sacfei.git
    (.nasaenv) cd sacfei/
    (.nasaenv) python setup.py build && pip install .
    ​
    # Install pymorph3 from github
    (.nasaenv) cd ~/tmp/
    (.nasaenv) git clone https://github.com/jgrss/pymorph3.git
    (.nasaenv) cd pymorph3/
    (.nasaenv) python setup.py build && pip install .
    ​
    # Install pytuyau from github
    (.nasaenv) cd ~/tmp/
    (.nasaenv) git clone https://github.com/jgrss/pytuyau.git
    (.nasaenv) cd pytuyau/
    (.nasaenv) python setup.py build && pip install .
    ​
    # Install IPython
    (.nasaenv) pip install ipython[all]
    ​
    # Deactivate the virtual environment when finished
    (.nasaenv) deactivate

**to uninstall and reinstall a package from a github repo:** (i.e. if changes are made to the package that need to be incorporated)

    source .nasaenv/bin/activate
    (.nasaenv) pip uninstall <package> -y
    (.nasaenv) cd ~/tmp/<package directory>
    #(look up the name of the main branch in the GitHub repo). Here it is 'main', but it could be 'master'/'dev'/etc. 
    (.nasaenv) git pull origin main
    (.nasaenv) python setup.py build && pip install .
    (.nasaenv) deactivate

(configVim)=
\#\#\# Configure vim (optional)
Text documents are edited in vim, which is not the most intuitive tool to those unfamiliar with it. Here you can find commands and tips for [working with vim](vimCommands). You can improve your experience with vim by creating a personalized
`.vimrc` file in your HOME directory ((output -\> \~/.vimrc) to configure vim.

Here is an example of a `.vimrc` doc (you can clone this from /jad-cel/cel-sandbox/templates/home_vimrc.sh):

    # Add line numbers
    set number
    ​
    au BufNewFile,BufRead *.py
        \ set tabstop=4 |
        \ set softtabstop=4 |
        \ set shiftwidth=4 |
        \ set textwidth=79 |
        \ set expandtab |
        \ set autoindent |
        \ set fileformat=unix
    	
    highlight BadWhitespace ctermbg=red guibg=darkred
    au BufRead,BufNewFile *.py,*.pyw,*.pyx,*.pxd,*.c,*.h match BadWhitespace /\s\+$/
    ​
    set encoding=utf-8
    ​
    let python_highlight_all=1
    syntax on
    ​
    set cursorline
    ​
    set foldmethod=indent
    set foldlevel=99
    ​
    set ic

### Add a personalized .profile file (optional) {#add-a-personalized-profile-file-optional}

You can add a personalized `.profile` file in your HOME directory (output -\> \~/.profile).
This is useful if you want to create keyboard shortcuts (alias) or ensure global variables are properly set.

Here is an example of things you might want in a `.profile` doc (you can clone this from /jad-cel/cel-sandbox/templates/home_profile.sh):

    # color enhancement for the terminal window
    export PS1="\[\033[36m\]\u\[\033[m\]@\[\033[32m\]\h:\[\033[33;1m\]\w\[\033[m\]\$ "
    export CLICOLOR=1
    export LSCOLORS=ExFxBxDxCxegedabagacad
    ​
    #simpler file navigation:
    alias .1='cd ../'
    alias .2='cd ../..'
    alias .3='cd ../../..'
    ​
    # shortcut to list SLURM job status ([username] = your username)
    alias qs="squeue -u [username]"

    if [ -n "$BASH_VERSION" ]; then
      if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
      fi
    fi
    ​
    if [ -d "$HOME/bin" ] ; then
      PATH="$HOME/bin:$PATH"
    fi
    ​
    # global vars for GDAL
    export CPLUS_INCLUDE_PATH=/usr/include/gdal
    export C_INCLUDE_PATH=/usr/include/gdal
    export GDAL_DATA=/usr/share/gdal/2.2

(profileIssue)=
\#\#\# Sourcing the profile (optional; only do if you created a .profile above)
:::{warning}Your personalized profile may not function correctly due to an irregular issue in the bash_profile.
:::
You could fix this everytime you log in by sourcing the profile:

    source .profile

But better to make the adjustment permanant by adding a`.bash_profile` to your home directory (output -\> \~/.bash_profile) with the following language (you can clone this from /jad-cel/cel-sandbox/templates/home_bash_profile.sh):

    # .bash_profile
    # Get the aliases and functions
    if [ -f ~/.bashrc ]; then
      . ~/.bashrc
      fi

    if [ -f ~/.profile ]; then 
      . ~/.profile
    fi

    # User specific environment and startup programs

    PATH=$PATH:$HOME/bin
    BASH_ENV=$HOME/.bashrc

    #export USERNAME BASH_ENV PATH
    export TMOUT=0

:::
:::