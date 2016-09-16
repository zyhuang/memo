# Python3 installation from scratch (without root permission)

Assuming packages will be installed to a customized directory, `PREFIX_DIR`, which will contain subdirs like `bin`, `lib`, `include`, `share`, etc. (e.g. `/usr/local`)

First off, create a file `PREFIX_DIR/share/config.site`, and add following two lines, which informs compilers about the customized flags during package installation (replace `<PREFIX_DIR>` with the real path).  

```bash
CPPFLAGS="-I<PREFIX_DIR>/include"
LDFLAGS="-L<PREFIX_DIR>/lib"
```

Next add `PREFIX_DIR/lib` to shell environment variable `LD_LIBRARY_PATH`, so that shared libraries installed at non-default locations can be easily linked during compilation. 

```bash
export LD_LIBRARY_PATH="<PREFIX_DIR>/lib":$LD_LIBRARY_PATH
```

Now install Python3 dependent packages:

* X11 related libraries [Tcl/Tk] (libtk, libtcl), [XscrnSaver] (libXss)
* compression libraries [bzip2] (libbz2), [xz] (liblzma)
* command line history [sqlite] (libsqlite3)
* terminal cursor [ncurse] (libncurses)

[Tcl/Tk](https://tcl.tk/software/tcltk/download.html)
[XscrnSaver]: http://www.linuxfromscratch.org/blfs/view/svn/x/x7lib.html
[bzip2]: http://www.bzip.org/downloads.html
[xz]: http://tukaani.org/xz/
[sqlite]: https://sqlite.org/download.html
[ncurse]: https://www.gnu.org/software/ncurses/



Compilation of these packges is fairly straightforward, following `configure --prefix=PREFIX_DIR (--enable-shared) && make && make install`. Tcl should be compiled before Tk. 

Now compile [Python3] (currently Python-3.5.2). It is a good practice that the package is built in a separate directory, `BUILD_DIR`, hence the code in the source directory is not messed up, and different builds can be made in parallel. Run the following to generate the Makefile.
[Python3]: https://www.python.org/downloads/

```bash
cd Python-3.5.2
mkdir build
cd build
../configure --prefix=<PREFIX_DIR>
```

If configuration goes through, before `make`, you might need to modify the `Python-3.5.2/setup.py` to help the compiler find the header files and libraries that you have just installed in `PREFIX_DIR`. Backup `setup.py` and add following lines to the `detect_modules()` method:

```python
add_dir_to_list(self.compiler.library_dirs, '<PREFIX_DIR>/lib')
add_dir_to_list(self.compiler.include_dirs, '<PREFIX_DIR>/include')
add_dir_to_list(self.compiler.include_dirs, '<PREFIX_DIR>/include/ncurses')
```

These edits will save your time with the following errors. 

```bash
../Include/py_curses.h:48:21: fatal error: ncurses.h: No such file or directory
...
*** WARNING: renaming "_bz2" since importing it failed: libbz2.so.1.0: cannot open shared object file: No such file or directory

Python build finished successfully!
The necessary bits to build these optional modules were not found:
_lzma                 _tkinter                                 
To find the necessary bits, look in setup.py in detect_modules() for the module's name.
Following modules built successfully but were removed because they could not be imported:
_bz2      
```

As everything is ready, now run `make`, and if successful, `make install`. If any missing header files or failure of import happens, go back and install the libraries (with shared library `.so` file) and redo the `make` until success. In bocca al lupo!


