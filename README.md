# PyPI howto

## Executive summary

- You need to create `~/.pypirc` and a `setup.py` for your project (examples below).
- You need to (pip) install `twine`.
- Check yourself before you wreck yourself: try first on https://test.pypi.org.
- If you made a mistake, you cannot reuse the same name-version combination, therefore
  it's good to practice first on https://test.pypi.org, or use e.g. `1.0.0-rc-1`, `1.0.0-rc-2`,
  and delete these versions afterwards before announcing `1.0.0`.


## Example `~/.pypirc`

```
[distutils]
index-servers=
    pypi
    pypitest

[pypi]
username = ...
password = ...

[pypitest]
repository = https://test.pypi.org/legacy/
username = ...
password = ...
```


## Upload to https://test.pypi.org

```
$ python setup.py sdist
$ twine upload dist/* -r pypitest
```

Once you are done with this step test to pip install from https://test.pypi.org:

```
$ pip install --index-url https://test.pypi.org/simple/ your-package
```


## Upload to https://pypi.org

```
$ python setup.py sdist bdist_wheel --universal
$ twine upload dist/* -r pypi
```

Run `setup.py` with `sdist` to build a source distribution and `bdist_wheel` to build a wheel (with `--universal` flag if your package is Python 2/3 universal). Then remove old versions from `/project/dist/` and ultimately use `twine` to upload to pypi.

## Release your code on GitHub

To tag your current commit as a released version, run:

```shell
git tag -a v1.0.0 -m "annotation for this release"
git push origin --tags
```

## Including other files

Python files not located in a submodule (i.e. a directory with an
`__init__.py`) or other files (not a `.py` file) can be included via a
`MANIFEST.in` file with one of two directives:

* `include` - used for single files or globbing files from a directory with `*`
* `recursive-include` - used for recursively adding files under a directory

After creating `MANIFEST.in` in your package directory add `include_package_data=True` to the `setup` object in `setup.py` (see below).


## Example `MANIFEST.in`

```
include doc/source/*rst
recursive-include project_name/data *
```


## Example `setup.py`

```python
from setuptools import setup
import os
import sys

if sys.version_info[0] < 3:
    with open('README.rst') as f:
        long_description = f.read()
else:
    with open('README.rst', encoding='utf-8') as f:
        long_description = f.read()

version = {}
with open(os.path.join('numerov', 'version.py')) as f:
    exec(f.read(), version)

setup(
    name='numerov',
    version=version['__version__'],
    description='Compute vibrational levels, wavefunctions, and expectation values using the Numerov-Cooley algorithm.',
    long_description=long_description,
    author='Radovan Bast',
    author_email='radovan.bast@uit.no',
    url='https://github.com/bast/numerov',
    license='MPL-2.0',
    packages=['numerov'],
    install_requires=[
        'click==6.7',
        'numpy==1.13.1',
        'pyyaml==3.12',
    ],
    scripts=['bin/cooley'],
    include_package_data=True,
    classifiers=[
        'Development Status :: 3 - Alpha',
        'Intended Audience :: Science/Research',
        'Programming Language :: Python :: 2.7',
        'Programming Language :: Python :: 3.6',
    ],
)
```

You can use the `setuptools` function [`find_packages()`](https://pythonhosted.org/setuptools/setuptools.html#using-find-packages)
to [automatically detect all subpackages and submodules](https://stackoverflow.com/a/14553799/943773), so long as they contain and `__init__.py` file.

```python
from setuptools import setup, find_packages

setup(
    #...
    packages=find_packages(),
)
```

## Tips

- If you wish to use `README.md` instead of `README.rst`, specify `long_description_content_type='text/markdown'` in `setup.py`.

- For automating the creation of `setup.py` files, see the documentation for
  [`setup.cfg`](https://setuptools.readthedocs.io/en/latest/setuptools.html#configuring-setup-using-setup-cfg-files).


## Credits

Thanks to [Ryan](https://github.com/ryanjdillon) for showing me the ropes.


## References

- https://caremad.io/posts/2013/07/setup-vs-requirement/
- https://gist.github.com/crccheck/3794099/
