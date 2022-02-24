# Autotools usage

## How to create a project

1. Create a file called configure.ac at your project's root directory:

```sh
mkdir my-project && cd my-project
touch configure.ac
```

NOTE: Previous versions of Autoconf promoted the name `configure.in`, which is somewhat ambiguous (the tool needed to process this file is not described by its extension), and introduces a slight confusion with config.h.in and so on (for which '.in' means "to be processed by configure"). Using `configure.ac` is now preferred, while the use of `configure.in` will cause warnings from autoconf.

1. Add to following template into `configure.ac`:

```
AC_INIT([my-project], [0.1],
        [http://github.com/aldoestebanpaz/my-project])

dnl Add support for Automake.
AM_INIT_AUTOMAKE

dnl Add support for Libtool.
LT_INIT

dnl Determine a C compiler to use.
AC_PROG_CC

dnl Create a C header file containing '#define' directives.
AC_CONFIG_HEADERS([config.h])

dnl An enhanced version of AC_PATH_X, which try to locate the X Window System include files and libraries.
dnl Adds the C compiler flags that X needs to output variable X_CFLAGS, and the X linker flags to X_LIBS.
AC_PATH_XTRA

dnl If AC_PATH_XTRA fails or the user gave the command line option --without-x,
dnl set the shell variable no_x to 'yes'; otherwise set it to the empty string.
if test x$no_x = xyes ; then
  AC_MSG_ERROR([X11 development libraries/headers required])
fi

dnl Ask for the creation of the following output files with the expansion of the output variables:
dnl     Makefile.in                => Makefile
dnl     myproject/Makefile.in      => myproject/Makefile
dnl     libmyproject/Makefile.in   => libmyproject/Makefile
dnl     test/Makefile.in           => test/Makefile
dnl     doc/Makefile.in            => doc/Makefile
dnl     libmy-project-0.1.pc.in    => libmy-project-0.1.pc
AC_CONFIG_FILES([
Makefile
myproject/Makefile
libmyproject/Makefile
test/Makefile
doc/Makefile
libmy-project-0.1.pc
])

AC_OUTPUT
```

NOTE 1: Every `configure.ac` must contain a call to `AC_INIT` before the checks, and a call to `AC_OUTPUT` at the end.

NOTE 2: In configure.ac, lines commented with '#' that occur after AC_INIT will appear in the resulting configure script. 'dnl' comments will not. Comments in Makefile.am are treated differently. Makefile.am is not processed by m4, but by automake, where the convention is to discard lines which begin with '##' preceded only by whitespace.

NOTE 3: For the header file specified in AC_CONFIG_HEADERS, with the appropriate -I option, you can use the '#include <config.h>' syntax. Actually, it’s a good habit to use it, because in the rare case when the source directory contains another `config.h`, the build directory should be searched first.

NOTE 4: The PKG_CHECK_MODULES macro is third-party macro installed with the `pkg-config` package (see `man pkg-config`).





1. Create the Makefile.am file.
2. Run `autoreconf --install` to install required files (auxiliary scripts and m4 macros) and create Makefile.in (same as `autoreconf` creates everyting else), plus aux-scripts. Example output:

```
libtoolize: putting auxiliary files in AC_CONFIG_AUX_DIR, 'aux-scripts'.
libtoolize: copying file 'aux-scripts/ltmain.sh'
libtoolize: putting macros in AC_CONFIG_MACRO_DIRS, 'm4'.
libtoolize: copying file 'm4/libtool.m4'
libtoolize: copying file 'm4/ltoptions.m4'
libtoolize: copying file 'm4/ltsugar.m4'
libtoolize: copying file 'm4/ltversion.m4'
libtoolize: copying file 'm4/lt~obsolete.m4'
configure.ac:19: installing 'aux-scripts/ar-lib'
configure.ac:19: installing 'aux-scripts/compile'
configure.ac:23: installing 'aux-scripts/config.guess'
configure.ac:23: installing 'aux-scripts/config.sub'
configure.ac:14: installing 'aux-scripts/install-sh'
configure.ac:14: installing 'aux-scripts/missing'
MyLibrary/Makefile.am: installing 'aux-scripts/depcomp'
```

4. Once you have Makefile.in files, you could run `autoconf` any time you modify configure.ac. If you modify a Makefile.am, then you have to run `autoreconf`.
5. Run `./configure`.
6. Run `make`.