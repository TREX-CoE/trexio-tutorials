
# TREXIO tutorials

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/TREX-CoE/trexio-tutorials/HEAD)

The goal of this repository is to provide material for new users of the TREXIO library in general 
and of the `trexio` Python API in particular.

To obtain a local copy of the `.ipynb` files, you can
[clone this repository](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/cloning-a-repository) 
and manually convert notebooks using, e.g. `jupytext --to notebook tutorial_benzene.md` command.


## Content

1. [Learn how to write and read data using the `trexio` Python API](notebooks/tutorial_benzene.md): the basics demonstrated by writing and reading some data corresponding to benzene molecule.
2. [Learn how to write and read the CI determinants and their coefficients using the `trexio` Python API](notebooks/how_to_determinants.md): the advanced use-case demonstrated by writing and reading the multi-determinant wave function components with arbitrary values.

### Why Jupyter Notebooks?


 * Jupyter notebooks are a common format for communicating scientific 
   information.
 * Jupyter notebooks can be launched in [Binder](https://www.mybinder.org), so that users can interact
   with tutorials.


### Note

You may notice that the notebooks are stored in markdown format (`.md` files) 
instead of the conventional `.ipynb` format. 
Conversion between `.ipynb` and `.md` notebook formats is facilitated by the 
[jupytext](https://jupytext.readthedocs.io/en/latest/index.html) package.

