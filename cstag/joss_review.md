# [betteridiot](https://github.com/betteridiot) JoSS review of [`cstag`](https://github.com/akikuno/cstag) and [`cstag-cli`](https://github.com/akikuno/cstag-cli)

Submitting author: @akikuno (Akihiro Kuno )<br/>
Repository: https://github.com/akikuno/cstag<br/>
Version: v3.2.2<br/>
Editor: @jbloom<br/>
Reviewer: @betteridiot<br/>

---
# Quick Checks:
1. General Stuff:
    - [x] Software was available at the listed repo
    - [x] MIT license present
    - [x] Version (v1.0.4) **did not** match the repo (v1.0.5)
    - [x] Authorship ([@akikuno](https://github.com/akikuno)) was appropriate
2. Documentation:
    - [ ] Statement of need was present and appropriate
    - [ ] Example usage was present (see **Notes** below)
3. Software paper:
    - [ ] Authors and affiliations present and appropriate
    - [ ] Statement of need also present within the paper
        
#### **Notes** on Quick Check items:
This is a minor thing that could lead people astray. The `CS` (uppercase) is a reserved for color read sequence information as per the [SAM format](https://samtools.github.io/hts-specs/SAMtags.pdf), while the `cs` (lowercase)
tag represents the difference string. This has already been a [topic of discussion](https://github.com/samtools/samtools/issues/1030#issuecomment-477137623) while feature requests to incorporate the `cs` tag more 
formallying into the SAM format.

---
# Installation:
Below are the steps I used to setup my environment for this review:

## Bioconda build
```bash
# Mark my current environment
$ conda info
    ...
    conda version: 23.10.0
    ...
    python version: 3.11.6.final.0
    ...
    platform: linux-64
 
$ conda create -n joss_cstag --no-default-packages --solver libmamba bioconda::cstag bioconda::cstag-cli
$ conda activate joss_cstag
```
:heavy_check_mark: Bioconda installation worked as expected.

## GitHub install
```bash
# Clean enviornment
$ conda remove -n joss_cstag --all
$ conda create -n joss_cstag --no-default-packages python
$ conda activate joss_cstag

# Get cstag and cstag-cli
$ git clone https://github.com/akikuno/cstag.git
$ git clone https://github.com/akikuno/cstag-cli.gi
$ cd cstag
$ python -m pip install . # passes

$ cd ../cstag-cli
$ python -m pip install . # Fails
...
checking for zlib.h... no
checking for inflate in -lz... no
configure: error: zlib development files not found

HTSlib uses compression routines from the zlib library <http://zlib.net>.
Building HTSlib requires zlib development files to be installed on the build
machine; you may need to ensure a package such as zlib1g-dev (on Debian or
Ubuntu Linux) or zlib-devel (on RPM-based Linux distributions or Cygwin)
is installed.

FAILED.  This error must be resolved in order to build HTSlib successfully.
...
error: subprocess-exited-with-error

× Getting requirements to build wheel did not run successfully.
│ exit code: 1
╰─> See above for output.

note: This error originates from a subprocess, and is likely not a problem with pip.
```
* :heavy_check_mark: Installation of `cstag` from `cstag/pyproject.toml` worked as intended.
* :triangular_flag_on_post: Installation of `cstag-cli` from `cstag-cli/pyproject.toml` **did not** work as intended.
    * The issue comes from the requirement of `pysam` which requires `htslib` which requires `zlib.h`. These dependencies are managed well by `conda` through Bioconda, but cannot be installed as an editable
    repository. While not documented as an intended installation option, I believe that it is important to note. This dependency issue was one of the principal reasons that I wrote `bamnostic`. It isn't a
    wrapper for `htslib`, so it has no requirements for it. This also allows it to be installed on any system that can have Python. This isn't a sales pitch though and, by no means, an indictment on `htslib`
    or `pysam`. These are terrific packages that serve the community/field in an essential way. But, as of [`htslib` v1.18](https://anaconda.org/bioconda/htslib) on Bioconda, Windows support is still missing.
    * More importantly, when regarding this review, `cstag-cli`'s dependencies to `pysam` (and `htslib` by assocation) prevent editable installs or installs on systems whereas the user does not have sudo privileges or access
    to `conda`/`mamba` since they cannot install `zlib.h` (easily) without them. Just something to consider.

## `pip` build
```bash
$ conda remove -n joss_cstag --all
$ conda create -n joss_cstag --solver libmamba --no-default-packages python
$ conda activate joss_cstag
$ pip install cstag
$ pip install cstag-cli
...
checking for zlib.h... no
checking for inflate in -lz... no
configure: error: zlib development files not found

HTSlib uses compression routines from the zlib library <http://zlib.net>.
Building HTSlib requires zlib development files to be installed on the build
machine; you may need to ensure a package such as zlib1g-dev (on Debian or
Ubuntu Linux) or zlib-devel (on RPM-based Linux distributions or Cygwin)
is installed.

FAILED.  This error must be resolved in order to build HTSlib successfully.
...
error: subprocess-exited-with-error

× Getting requirements to build wheel did not run successfully.
│ exit code: 1
╰─> See above for output.

note: This error originates from a subprocess, and is likely not a problem with pip.

$ conda install --solver libmamba conda-forge::zlib && pip install cstag-cli
```
:triangular_flag_on_post: It appears the direct `pip` install from PyPI has the same issue as above regarding installation dependencies (not surprising). By installing `zlib` through `conda-forge` on a system that does not have `zlib.h` available,
one can overcome this issue and successfully install `cstag-cli`.

This dependency issue does not rest with your program in any way. It is a product of `pysam`/`htslib` dependencies. I also acknowledge that most people in bioinformatics don't often has as sterile of an environment as I do. 
I just suspect that there might be some graduate student out there that has only lightly used `conda`, `pip`, or any other package manager could potentially be stumped by this. 

My suggestion is to make the `pysam` (and, by extension, `htslib`) dependencies more explicitly documented on your repo/`README.md` since these dependencies cannot be address by `pip` alone. However, they are properly managed
by `conda`/`mamba`. While an edge case, it is still a case.

From this point on, I will be using the [GitHub build](#github-install) for Bioconda build for review and the GitHub build for testing.

---

# Testing the general import

```bash
$ python -c "import cstag; print(True)"
True
$ cstag --help
```
:heavy_check_mark: Basic import of `cstag` ran without error.
:heavy_check_mark: Help prompt of `cstag-cli` ran without error.

---
# Testing with `pytest`
:triangular_flag_on_post: Missing documentation on testing the build in `README.md`.
:triangular_flag_on_post: Tests not included in package installs. Consider adding `tests` to your [`pyproject.toml`](https://python-poetry.org/docs/pyproject/#include-and-exclude)
    * Will be using `GitHub
 
* ```bash
# Run test suite
$ conda install --solver libmamba pytset
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
