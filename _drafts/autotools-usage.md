# Autotools usage

## Downloading, configuring, making and installing a distributable package (common style)

Example with hello-2.12.tar.gz:

```sh
# Download
wget -O hello-2.12.tar.gz "https://ftp.gnu.org/gnu/hello/hello-2.12.tar.gz"

# Unpack
tar zxf hello-2.12.tar.gz


# Configure
cd hello-2.12
./configure
# ...
# config.status: creating Makefile
# config.status: creating src/Makefile
# ...
echo $?
# 0


# Run Make (shorthand for `make all`)
make
# ...


# Run tests on the generated files in the source tree
make check
# ...


# Install (shorthand for `make install-exec install-data`)
su
make install
# ...
exit


# Run tests on the installed files
make installcheck
```

By default everything will be installed in subdirectories of '/usr/local':
- binaries will go into '/usr/local/bin',
- ibraries will end up in '/usr/local/lib', etc.
This destination is usually not writable by any user, so we assume that we have to become root before we can run `make install`. In our example, running `make install` will copy many files like the program hello into '/usr/local/bin' and hello.1 into '/usr/local/share/man/man1'.

`make installcheck` run tests on the installed files. `make check` tests the files in the source tree, while `make installcheck` tests their installed copies.

## Configuring, and making distributable package using VPATH builds (cmake style)

A common request from users is that they want to confine all derived files to a single directory, to keep their source directories uncluttered. Here is how we could run configure to create everything in a build tree (that is, subdirectory) called 'build'.

Example with hello-2.12.tar.gz:

```sh
# Configure
cd hello-2.12
mkdir build && cd build
../configure
# ...
# config.status: creating Makefile
# config.status: creating src/Makefile
# ...
echo $?
# 0


# Run Make (same as `make all`)
make
# ...
```

## Installing in non-default directories

(1) **determines where the package will finally go when it is installed, and where it will look for its associated files when it is run**. It's what you should use if you're just compiling something for use on a single host.

(2) **is for installing to a temporary directory which is not where the package will be run from**. For example this is used when building deb packages. When a person/system builds a binary package, he doesn't actually install everything into its expected final place (maybe configured with (1)) on his own system. He may have a different version installed already and not want to disturb it, or he may not even be root.

### (1) Using './configure --prefix ...'

NOTE: The value of --prefix should be an absolute directory name.

```sh
./configure --prefix /usr
make
make install
```

This would install '/usr/bin/hello', '/usr/share/man/man1/hello.1' and other files.

### (2) Using 'make install DESTDIR=...'

```sh
./configure
make
make DESTDIR=${PWD}/tmp install
```

### Combining both

As explained before './configure --prefix ...' configures the package for final installation, 'make install DESTDIR=...' installs the package into a temporal directory generally for binary-package creation. See 'Building Binary Packages using DESTDIR (staged installations)' for an example.

### Changins installation locations with 'make' and 'make install'

**The most portable way to affect installation locations is to pass the correct locations to configure**; however, many packages provide one or both of the following shortcuts of passing variable assignments to the 'make' and 'make install' command lines to change installation locations without having to reconfigure.

Example:

```sh
make clean

# build
make prefix=/usr/local

# install
make prefix=/usr/local DESTDIR=${PWD}/tmp install
```

It will install the binaries on '${PWD}/tmp/usr/local/bin'.

Those directories may be stored in the compiled binaries, so we have to compile the package and install it with the same configuration.

The variables in the Makefile will follow the same ./configure convention, and the prefix variable is used as the base directory for other parameters.

## Standard directory variables

The GNU Coding Standards also specify a hierarchy of variables to denote installation directories. Some of these are:

| Directory variable | Default value                 |
|--------------------|-------------------------------|
| prefix             | /usr/local                    |
| exec_prefix        | ${prefix}                     |
| bindir             | ${exec_prefix}/bin            |
| libdir             | ${exec_prefix}/lib            |
| includedir         | ${prefix}/include             |
| datarootdir        | ${prefix}/share               |
| datadir            | ${datarootdir}                |
| mandir             | ${datarootdir}/man            |
| infodir            | ${datarootdir}/info           |
| docdir             | ${datarootdir}/doc/${PACKAGE} |

Each of these directories has a role which is often obvious from its name. In a package, any installable file will be installed in one of these directories.

For instance,
- The program 'hello' is to be installed in _bindir_, the directory for binaries. The default value for this directory is '/usr/local/bin'.
- A README will be installed into _docdir_, which defaults to '/usr/local/share/doc/<PACKAGE NAME>'

**The user can supply a different value when calling 'configure'**.

In the list of directory variables provided earlier, all the variables based on exec-prefix designate architecture-dependent directories whose files will be installed by `make install-exec`. The others designate architecture-independent directories and will serve files installed by `make install-data`. See [The Two Parts of Install](https://www.gnu.org/software/automake/manual/automake.html#The-Two-Parts-of-Install), for more details.

**`make install`, can be thought of as a shorthand for `make install-exec install-data`.**.

## The default target

**`make` is a shorthand for `make all`, 'all' being the default target in the GNU Build System and in Automake-generated Makefiles. But it does not need to be the default in third-party Makefiles**.

`make all` build programs, libraries, documentation, etc.

## Source tree vs. Build tree and Parallel build trees (aka. VPATH builds)

The GNU Build System distinguishes two trees: the source tree, and the build tree. These are two directories that may be the same, or different.

- The source tree is rooted in the directory containing the configure script. It contains all the source files (those that are distributed), and may be arranged using several subdirectories.

- The build tree is rooted in the current directory at the time configure was run, and is populated with all object files, programs, libraries, and other derived files built from the sources (and hence not distributed). The build tree usually has the same subdirectory layout as the source tree.

**If 'configure' is executed in its own directory, the source and build trees are combined: derived files are constructed in the same directories as their sources**.

A setup where source and build trees are different, are often called 'parallel builds' or 'VPATH builds'. The expression 'parallel build' is misleading: the word 'parallel' is a reference to the way the build tree shadows the source tree, it is not about some concurrency in the way build commands are run. For this reason we refer to such setups using the name 'VPATH builds'.

'VPATH' is the name of the make feature used by the Makefiles to allow these builds.

## Creating a distributable file

`make dist` collects all your source files and the necessary parts of the build system to create a tarball named 'package-version.tar.gz'.

The 'dist' rule in the generated 'Makefile.in' can be used to generate a gzipped tar file and/or other flavors of archives for distribution. The file is named based on the PACKAGE and VERSION variables automatically defined by either the AC_INIT invocation or by a deprecated two-arguments invocation of the AM_INIT_AUTOMAKE macro. More precisely, the gzipped tar file is named '${PACKAGE}-${VERSION}.tar.gz'.

### Files automatically distributed

For the most part, the files to distribute are automatically found by Automake:

- All source files
- All Makefile.am and Makefile.in files.
- Files that are read by 'configure'. These are the source files as specified in various Autoconf macros such as AC_CONFIG_FILES and siblings.
- Files included in a Makefile.am (using include) or in configure.ac (using m4_include).

#### Automake's built-in lists of files to be automatically distributed

The following are built-in lists of files that automake fill include automatically too:

- Commonly used: these files are included if they are found in the current directory (either physically, or as the target of a Makefile.am rule). Some common examples: ABOUT-GNU, COPYING, TODO.
This list also includes helper scripts installed with `automake --add-missing`. Some common examples: compile, config.guess, config.rpath, config.sub, texinfo.tex.
- Plain name or with .md extension: automatically distributed if they are found either with the plain name, or with extension .md (presumably MarkDown, though this not checked). They are checked for in that order, so the plain name is preferred. These are: AUTHORS ChangeLog INSTALL NEWS README README-alpha THANKS.
- Under certain conditions: files distributed only if other certain conditions hold. For example, the files config.h.top and config.h.bot are automatically distributed only if, e.g., 'AC_CONFIG_HEADERS([config.h])' is used in configure.ac). README-alpha is another such file, with README-alpha.md distributed if that is what is available; see [Strictness](https://www.gnu.org/software/automake/manual/automake.html#Strictness), for its conditions for distribution.

These three lists of files are given in their entirety in the output from `automake --help`.

### Including other files

Despite all this automatic inclusion, it is still common to have files to be distributed which are not found by the automatic rules. You should list these files in the EXTRA_DIST variable. You can mention files in subdirectories in EXTRA_DIST.

You can also mention a directory in EXTRA_DIST; in this case the entire directory will be recursively copied into the distribution. NOTE: To emphasize, this copies everything in the directory, including temporary editor files, intermediate build files, version control files, etc.; thus we recommend against using this feature as-is. However, you can use the dist-hook feature to ameliorate the problem.

If you define SUBDIRS, Automake will recursively include the subdirectories in the distribution. If SUBDIRS is defined conditionally, Automake will normally include all directories that could possibly appear in SUBDIRS in the distribution. If you need to specify the set of directories conditionally, you can set the variable DIST_SUBDIRS to the exact list of subdirectories to include in the distribution. See [Conditional Subdirectories](https://www.gnu.org/software/automake/manual/automake.html#Conditional-Subdirectories) for more information.

#### SUBDIRS vs DIST_SUBDIRS

SUBDIRS contains the subdirectories of the current directory that must be built. It must be defined manually; Automake will never guess a directory is to be built. As we will see in the next two sections, it is possible to define it conditionally so that some directory will be omitted from the build.

DIST_SUBDIRS is used in rules that need to recurse in all directories, even those that have been conditionally left out of the build. Recall our example where we may not want to build subdirectory opt/, but yet we want to distribute it? This is where DIST_SUBDIRS comes into play: 'opt' may not appear in SUBDIRS, but it must appear in DIST_SUBDIRS.

Precisely, DIST_SUBDIRS is used by `make maintainer-clean`, `make distclean` and `make dist`. All other recursive rules use SUBDIRS.

If SUBDIRS is defined conditionally using Automake conditionals, Automake will define DIST_SUBDIRS automatically from the possible values of SUBDIRS in all conditions.

If SUBDIRS contains AC_SUBST variables, DIST_SUBDIRS will not be defined correctly because Automake does not know the possible values of these variables. In this case DIST_SUBDIRS needs to be defined manually.

#### The dist Hook

Occasionally it is useful to be able to change the distribution before it is packaged up. If the dist-hook rule exists, it is run after the distribution directory is filled, but before the actual distribution archives are created. See [The dist Hook](https://www.gnu.org/software/automake/manual/automake.html#The-dist-Hook) for more information.

## Building Binary Packages using DESTDIR (staged installations)

The GNU Build System does not replace a package manager.

Such package managers usually need to know which files have been installed by a package, so a mere `make install` is inappropriate.

The DESTDIR variable can be used to perform a staged installation. The package should be configured as if it was going to be installed in its final location (e.g., `--prefix /usr`), but when running `make install`, the DESTDIR should be set to the absolute name of a directory into which the installation will be diverted. From this directory it is easy to review which files are being installed where, and finally copy them to their final location by some means.

Example creating a binary package containing a snapshot of all the files to be installed:

```sh
# Configure the package for final installation
./configure --prefix /usr
#...


# Build
make
# ...


# Install into DESTDIR
make DESTDIR=$HOME/inst install
# ...


# Create the binary package
cd ~/inst
find . -type f -print > ../files.lst
tar zcvf ~/hello-2.12-i686.tar.gz  `cat ../files.lst`
# ...
```

After this example, hello-2.12-i686.tar.gz is ready to be uncompressed in / on many hosts. (Using `cat ../files.lst` instead of `.` as argument for tar avoids entries for each subdirectory in the archive: we would not like tar to restore the modification time of /, /usr/, etc.).

## Tests

### 'check'

`make check` causes the package's tests to be run. This step is not mandatory, but it is often good to make sure the programs that have been built behave as they should, before you decide to install them. **If the package does not contain any tests, so running `make check` is a no-op**.

### 'installcheck'

`make installcheck` run tests on the installed files. It can make a difference, for instance when the source tree's layout is different from that of the installation. Furthermore it may help to diagnose an incomplete installation.

**`make check` tests the files in the source tree, while `make installcheck` tests their installed copies**. The tests run by the latter can be different from those run by the former. For instance, there are tests that cannot be run in the source tree.

Some packages are set up so that `make installcheck` will run the very same tests as `make check`, only on different files (non-installed vs. installed). **Presently most packages do not have any installcheck tests because the existence of installcheck is little known, and its usefulness is neglected**.

## Cleaning generated files

Erase from the build tree the files built by `make` (aka. `make all`):

```sh
make clean
```

Calls `make clean` and additionally erase anything `./configure` created, including the Makefile's:

```sh
make distclean
```

Erase the installed files (the opposite of `make install`). NOTE: this needs to be run from the same build tree that was installed:

```sh
make uninstall
```

### 'distcheck'

`make distcheck` constructs 'package-version.tar.gz' (just as well as the 'dist' target) and additionally ensures the following use cases work:

(1) Tries to do a 'VPATH build' with the 'srcdir' (the source tree) and all its content made read-only.

(2) Tries a full compilation of the package: it implies ...
- unpacking the newly constructed tarball,
- running `make` to compile the code,
- `make dvi` to make the printable documentation, if any,
- `make check` to run the test suite on this fresh build,
- `make install` installs the package in a temporary directory,
- `make installcheck` to run the test suite on the resulting installation
- `make dist` to make another tarball to ensure the distribution is self-contained.

(3) It also makes sure the following targets do not omit any file ...
- it checks the package can be correctly cleaned with `make clean` and `make distclean`
- it checks the package can be correctly uninstalled with `make uninstall`

(4) It checks that DESTDIR installations work.

All of these actions are performed in a temporary directory, so that no root privileges are required. The exact location and the exact structure of such a subdirectory (where the extracted sources are placed, how the temporary build and install directories are named and how deeply they are nested, etc.) is to be considered an implementation detail, which can change at any time; so do not rely on it.

**Releasing a package that fails `make distcheck` means that one of the scenarios we presented will not work and some users will be disappointed. Therefore it is a good practice to release a package only after a successful `make distcheck`**.

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

NOTE 3: For the header file specified in AC_CONFIG_HEADERS, with the appropriate -I option, you can use the '#include <config.h>' syntax. Actually, it's a good habit to use it, because in the rare case when the source directory contains another `config.h`, the build directory should be searched first.

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
