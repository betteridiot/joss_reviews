# [betteridiot](https://github.com/betteridiot) JoSS review of [Hickle](https://github.com/telegraphic/hickle)

Submitting author: @telegraphic (Danny Price)<br/>
Repository: https://github.com/telegraphic/hickle<br/>
Version: v3.2.2<br/>
Editor: @arfon<br/>
Reviewer: @betteridiot<br/>

---
# Quick Checks:
1. General Stuff:
    * Software was available at the listed repo
    * MIT license present
    * Version (3.2.2) matched the repo
    * Authorship (@telegraphic) was appropriate
2. Documentation:
    * Statement of need was present and appropriate
    * Example usage was present (see **Notes** below)
3. Software paper:
    * Authors and affiliations present and appropriate
    * Statement of need also present within the paper
    * Advised author to add a DOI to paper.bib [here](https://github.com/telegraphic/hickle/issues/79) 
        * The author addressed this change in https://github.com/telegraphic/hickle/commit/304cb5cdbdf40b136d5764d3121d623ddbfcc6b6
    * Advised author to add contibution guidelines [here](https://github.com/telegraphic/hickle/issues/80). 
        * The author addressed this in https://github.com/telegraphic/hickle/commit/ca2a61225d0233ffa6f2fb0c61dc81d911bac5f6
        
#### **Notes** on Quick Check items:
This is more of a personal quibble, and @arfon should probably arbitrate this: while `hickle` is intended as a 'drop-in' replacement for `pickle`, it is actually an amalgam of `h5py` and `pickle/dill`. This means it has *extended* functionality. The issue I have with this is that the documentations kind of glosses over the `h5py` functionality with a link pointing to their documentation. 

The reason this is an issue is that if `hickle` were to be a one stop shop for serializing to an HDF5 intermediate, I shouldn't have to go to someone else's documentation to do it. `hickle` is great! You should highlight this interoperability. 

Lastly, with regards to documentation, I understand that `pickle` is very well-known. I also understand that (for those that use HDF5) `pytables` and `h5py` are well known as well. But there is no module level documentation for `hickle`. By that, I mean there is no reference to class or method documentation (docstrings). 

I don't expect a [Read the Docs](https://readthedocs.org/) page for every package, but you already have the docstrings...a little time using [Sphinx autodoc](http://www.sphinx-doc.org/en/master/usage/extensions/autodoc.html) can make your package very accessible.

---
# Installation:
Below are the steps I used to setup my environment for this review:

```bash

# Mark my current environment
$ conda info
    ...
    conda version: 4.5.11
    ...
    python version: 3.7.0.final.0
    base environment: /user/<user>/home/miniconda3
    ...
    platform: linux-64

# Isolate my environment with `conda env`
$ conda create -n joss_base python=3.7 --no-default-packages
$ conda create -n joss_hickle --clone joss_base numpy=1.51.2 h5py dill
$ conda activate joss_hickle

# Isolate my review directory
$ mkdir -p <mount>/<user>/GitHub/joss_reviews/hickle
$ cd <mount>/<user>/GitHub/joss_reviews/hickle

# Get Hickle
$ git clone https://github.com/telegraphic/hickle.git
$ cd hickle

# Install as seen in README.md
$ python setup.py install

```

Everything worked as intended.

---
# Testing the general import

```bash

$ python -c "import hickle; print(True)"
True

```

Basic import ran without error

---
# Testing with `pytest`

```bash

# Run test suite
$ pytest tests/
# error: scipy not found

$ conda install scipy
$ pytest tests/
# error: astropy not found

$ conda install astropy
$ pytest tests/
# All tests passed

```

**Recommendation**: Add scipy and astropy to your `requirements.txt`

---
# Testing code coverage

```bash

$ pytest --cov
...
----------- coverage: platform linux, python 3.7.1-final-0 -----------
Name                           Stmts   Miss  Cover
--------------------------------------------------
tests/test_astropy.py            101     11    89%
tests/test_hickle.py             535    156    71%
tests/test_hickle_helpers.py      41      8    80%
tests/test_legacy_load.py         24     14    42%
tests/test_scipy.py               45      4    91%
--------------------------------------------------
TOTAL                            746    193    74%

```

Only thing to say here is that maybe `legacy_load` could use some help (more on that in the next section)

---
# Testing doctests

Because I saw docstrings, I figured there might be doctests. So I ran pytest on your docstrings and found some fun stuff:

```bash

$ pytest --doctest-modules

```

This caused all sorts of issues:
1. The docstrings mention `pandas`, and since it wasn't a requirement, it wasn't available and caused an error. 
    * Consider removing your references to `pandas` or just add it as a requirement.
2. 'exceptions' module not found
    * In `hickle_legacy`, `exceptions` is imported. While this worked perfectly fine on a Python 2.7 environment, it will never work in a Python 3.x one. The good news here is that you don't even need it. All the exceptions are automatically loaded into the builtin namespace. So if you remove the import and change your custom exception inheritance from `exceptions.Exception` to `Exception` this is no longer a problem.
3. 'unicode' not defined/'long' not defined/ImportError on 'NoneType':
    * These are all just py27 vs py3 issues. I fixed these just by doing some version checking and using some recasting. I will send a PR with what I did to make `pytest --doctest-modules` complete without error.

---
# Quickstart

I installed `PyTables` just to test the interoperability of your HDF5 intermediates.
```bash

conda install pytables

```

Then dropped into a console and went to work:

```python

>>> import numpy as np
>>> x = np.random.random((2000, 2000))

# Assess the standard
>>> import pickle as pkl
>>> with open('joss_hkl_test.pkl', 'wb') as pk_file: # Needed for Python 3.x
...     pkl.dump(x, pk_file)

# Compare with hickle
>>> import hickle as hkl
>>> with open('joss_hkl_test.hkl', 'w') as hk_file:
...     hkl.dump(x, hk_file)
# Working as intended

# Check file size difference
>>> import os
>>> hkl_size = os.stat('joss_hkl_test.hkl').st_size
>>> pkl_size = os.stat('joss_hkl_test.pkl').st_size
>>> hkl_size > pkl_size
True # Negligible though: ~6kb

# Testing HDF5 compatibility with PyTables
>>> import tables
>>> tables.open('joss_hkl_test.hkl')
File(filename=joss_hkl_test.hkl, title='', mode='r', root_uep='/', filters=Filters(complevel=0, shuffle=False, bitshuffle=False, fletcher32=False, least_significant_digit=None))
/ (RootGroup) ''
/data_0 (Array(2000, 2000)) ''
  atom := Float64Atom(shape=(), dflt=0.0)
  maindim := 0
  flavor := 'numpy'
  byteorder := 'little'
  chunkshape := None

```

Everything appears to be working as intended.
