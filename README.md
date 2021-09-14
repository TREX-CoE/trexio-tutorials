
# TREXIO tutorials

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/TREX-CoE/trexio-tutorials/HEAD)

The goal of this repository is to provide material for new users of the TREXIO library in general 
and of the Python API in particular.

To obtain a local copy of the `.ipynb` files, you can
[clone this repository](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/cloning-a-repository) 
and manually convert notebooks using, e.g. `jupytext --to notebook tutorial_benzene.md` command.


## Content

1. [Learn how to write and read data using TREXIO API](notebooks/tutorial_benzene.md): the basics demonstrated by writing and reading some data corresponding to benzene molecule.


### Why Jupyter Notebooks?


 * Jupyter notebooks are a common format for communicating scientific 
   information.
 * Jupyter notebooks can be launched in [Binder](https://www.mybinder.org), so that users can interact
   with tutorials.


### Note

You may notice that the notebooks are stored in markdown format (`.md` files) 
instead of conventional `.ipynb`. 
Conversion between `.ipynb` and `.md` notebook formats is facilitated by the 
[jupytext](https://jupytext.readthedocs.io/en/latest/index.html) package.

