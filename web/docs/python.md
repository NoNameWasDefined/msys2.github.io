# Python

Since the official CPython implementation doesn't support building with GCC/Clang on Windows and has its own Windows specific directory layout, we maintain a friendly fork of CPython at https://github.com/msys2-contrib/cpython-mingw/

Some differences/features compared to the official Windows CPython:

* In an active MSYS2 environment `os.sep` and `os.altsep` are switched to make relative paths more compatible with Unix tools that don't understand Windows paths. Outside of an active MSYS2 environment it behaves normally though.
* `sys.path` uses the Unix directory layout, see `python -m site`
* Virtual environments also work with bash: `python -m venv _venv`, `source _venv/bin/activate` and so on.
* When building against the limited API only defining `Py_LIMITED_API` isn't enough, you also have to explicitly link against `python3` instead of `python3.y`.

### Portability

As long as you don't hardcode/assume platform specific values and paths and always use things like `os.sep`, do path operations with `os.path` or `pathlib` and derive Python installation related paths and configuration from the `sysconfig` module then your Python code should work just like with the official Windows CPython installation.

If for some reason you still need to detect our fork you can check for it as follows:

```python
import os
import sysconfig

if os.name == "nt" and sysconfig.get_platform().startswith("mingw"):
    print("cpython-mingw detected!")
```

### Known issues

* C extensions are not compatible with the official CPython, which means pip can't use binary wheels from PyPI and packages have to be build when installing them.
* Some C extensions don't build out of the box since they don't expect non-MSVC on Windows. In some cases we provide patched versions in our repo.
* Passing `--py-limited-api` to upstream setuptools does not result in linking with the limited ABI DLL. See the [upstream issue](https://github.com/pypa/setuptools/issues/4224). The setuptools package in our repo is patched to fix this.

### Changelog

* 2024-11-09: With the update to Python 3.12, the environment variable setting `SETUPTOOLS_USE_DISTUTILS=stdlib` will cause distutils import errors because the standard library's distutils module has been removed. If you're still using this workaround, remove the environment variable.
* 2024-07-01: [setuptools](https://github.com/pypa/setuptools) now supports building C extensions in MSYS2 since [v70.2.0](https://github.com/pypa/setuptools/releases/tag/v70.2.0). Previous versions required `export SETUPTOOLS_USE_DISTUTILS=stdlib` as a workaround.
* 2023-08-22: The mingw Python now provides the limited ABI DLL (libpython3.dll). Support in upstream setuptools is still missing though.
