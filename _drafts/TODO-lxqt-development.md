- [Clone project](#clone-project)
- [Get latest changes](#get-latest-changes)
- [How to search dependency packages before build](#how-to-search-dependency-packages-before-build)
  - [How to search packages with pkg-config files](#how-to-search-packages-with-pkg-config-files)
  - [How to search packages with cmake packages](#how-to-search-packages-with-cmake-packages)
- [REQUIRED - Clear before a new build attempt](#required---clear-before-a-new-build-attempt)
- [How LXQT\_TRANSLATIONS\_DIR and LXQT\_ETC\_XDG\_DIR variables are being defined in the project](#how-lxqt_translations_dir-and-lxqt_etc_xdg_dir-variables-are-being-defined-in-the-project)
- [Build lxqt-runner](#build-lxqt-runner)
  - [Example commands](#example-commands)
  - [Outputs](#outputs)
    - [Output 1](#output-1)
    - [Output 2](#output-2)
    - [Output 4](#output-4)


## Clone project

```sh
git clone https://github.com/lxqt/lxqt.git
cd lxqt
git submodule init
git submodule update --remote --rebase
```

## Get latest changes

```sh
cd lxqt
git submodule update --remote --rebase
```

## How to search dependency packages before build

### How to search packages with pkg-config files

The packages which provide the *.pc configuration files have names of the form "<PACKAGENAME>-dev", e.g. The statement `pkg_check_modules(MUPARSER REQUIRED muparser)` tries to search the package "MUPARSER" (this means that a "muparser.pc" file should be included) that is included in the deb package "libmuparser-dev".

Use apt-file to search for the missing packages.

```sh
# Example
# Cmake command "pkg_check_modules(MUPARSER REQUIRED muparser)", Package file name: "muparser.pc"
apt-file search muparser.pc
# result:
#   libmuparser-dev: /usr/lib/x86_64-linux-gnu/pkgconfig/muparser.pc
```

### How to search packages with cmake packages

The packages which provide the *.cmake configuration files have names of the form "<PACKAGENAME>-dev", e.g. The cmake package "Qt5Widgets" (this means that a "Qt5WidgetsConfig.cmake" or "qt5widgets-config.cmake" package configuration file should be included) is included in the deb package "qtbase5-dev".

Use apt-file to search for the missing packages.

Examples

```sh
sudo apt-file update

# Example 1
# Package "Qt5Widgets", Package file name: "Qt5WidgetsConfig.cmake"
apt-file search Qt5WidgetsConfig.cmake
# result:
#   qtbase5-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5Widgets/Qt5WidgetsConfig.cmake
#   qtbase5-gles-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5Widgets/Qt5WidgetsConfig.cmake

# Example 2
# Package "lxqt-globalkeys-ui", Package file name: "lxqt-globalkeys-ui-config.cmake"
apt-file search lxqt-globalkeys-ui-config.cmake
# result:
#   liblxqt-globalkeys-ui0-dev: /usr/share/cmake/lxqt-globalkeys-ui/lxqt-globalkeys-ui-config.cmake
```

## REQUIRED - Clear before a new build attempt

After an unsuccessful build, you should remove CMakeCache.txt (or simply clear the build directory); otherwise cmake will report the same error even if the needed cmake package has been installed.

```sh
cd <PROJECT_DIR>/build
rm -rf *
```

## How LXQT_TRANSLATIONS_DIR and LXQT_ETC_XDG_DIR variables are being defined in the project

- The command `find_package(lxqt ...)` searchs for the cmake package file "/usr/share/cmake/lxqt/lxqt-config.cmake" (included in the package "liblxqt0-dev").
- "/usr/share/cmake/lxqt/lxqt-config.cmake" has the commands `include(CMakeFindDependencyMacro)` and `find_dependency(lxqt-build-tools ...)` ("find_dependency" is a wrapper around "find_package" designated to be used inside cmake package files).
- The command `find_dependency(lxqt-build-tools ...)` searchs for the cmake package file "/usr/share/cmake/lxqt-build-tools/lxqt-build-tools-config.cmake" (included in the package "lxqt-build-tools").
- "/usr/share/cmake/lxqt-build-tools/lxqt-build-tools-config.cmake" executes the following commands:
```cmake
set(LXQT_CMAKE_MODULES_DIR "${PACKAGE_PREFIX_DIR}/share/cmake/lxqt-build-tools/modules/")
set(LXQT_CMAKE_FIND_MODULES_DIR "${PACKAGE_PREFIX_DIR}/share/cmake/lxqt-build-tools/find-modules/")

list(APPEND CMAKE_MODULE_PATH
    "${LXQT_CMAKE_MODULES_DIR}"
    "${LXQT_CMAKE_FIND_MODULES_DIR}"
)
```
- The command `list(APPEND CMAKE_MODULE_PATH ...)` adds new lookup locations for modules.
- "/usr/share/cmake/lxqt/lxqt-config.cmake" has the command `include(LXQtConfigVars)`.
- The command `include(LXQtConfigVars)` searchs for the cmake module file "/usr/share/cmake/lxqt-build-tools/modules/LXQtConfigVars.cmake" (included in the package "lxqt-build-tools").
- "/usr/share/cmake/lxqt-build-tools/modules/LXQtConfigVars.cmake" executes the command `set(LXQT_TRANSLATIONS_DIR   "/usr/share/lxqt/translations")` and `set(LXQT_ETC_XDG_DIR        "/etc/xdg")` that defines the variable "LXQT_TRANSLATIONS_DIR" and "LXQT_ETC_XDG_DIR".
- "LXQT_TRANSLATIONS_DIR" is the default location where translation files will be installed.
- "LXQT_ETC_XDG_DIR" is the default location where autostart files will be installed.

## Build lxqt-runner

### Example commands

```sh
# Package "Qt5Widgets", Package file name: "Qt5WidgetsConfig.cmake"
# Package "Qt5Xml", Package file name: "Qt5XmlConfig.cmake"
sudo apt-get -y install qtbase5-dev
# Package "Qt5LinguistTools", Package file name: "Qt5LinguistToolsConfig.cmake"
sudo apt-get -y install qttools5-dev
# Package "KF5WindowSystem", Package file name: "KF5WindowSystemConfig.cmake"
sudo apt-get -y install libkf5windowsystem-dev
# Package "lxqt", Package file name: "lxqt-config.cmake"
# "liblxqt0-dev" installs the package "lxqt-build-tools" too. It includes de cmake module files:
# "LXQtPreventInSourceBuilds" (/usr/share/cmake/lxqt-build-tools/modules/LXQtPreventInSourceBuilds.cmake).
# and "LXQtCompilerSettings" (/usr/share/cmake/lxqt-build-tools/modules/LXQtCompilerSettings.cmake)
sudo apt-get -y install liblxqt0-dev
# Package "lxqt-globalkeys-ui", Package file name: "lxqt-globalkeys-ui-config.cmake"
sudo apt-get -y install liblxqt-globalkeys-ui0-dev
# Required by the cmake package "lxqt"
# Package "Qt5X11Extras", Package file name: "Qt5X11ExtrasConfig.cmake"
sudo apt-get -y install libqt5x11extras5-dev
# Required by the cmake package "lxqt-globalkeys-ui"
# Package "lxqt-globalkeys", Package file name: "lxqt-globalkeys-config.cmake"
sudo apt-get -y install liblxqt-globalkeys0-dev

# Cmake command "pkg_check_modules(MUPARSER REQUIRED muparser)", Package file name: "muparser.pc"
sudo apt-get -y install libmuparser-dev

cd lxqt/lxqt-runner

git checkout 0.17.0

mkdir build && cd build
rm -rf *
# CMAKE_BUILD_TYPE can specify a build (debug|release|...) build type
# LIB_SUFFIX can set the ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}
#     useful fro 64 bit distros
# CMAKE_INSTALL_PREFIX changes default /usr/local prefix
cmake \
  -DCMAKE_BUILD_TYPE=debug \
  -DCMAKE_INSTALL_PREFIX=/usr \
  -DLIB_SUFFIX="" \
  ..
# Output 1
make clean
make
# Output 2
make \
  DESTDIR=${PWD}/install \
  install
# Output 3
```

### Outputs

#### Output 1

```
-- The C compiler identification is GNU 11.3.0
-- The CXX compiler identification is GNU 11.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found X11: /usr/include   
-- Looking for XOpenDisplay in /usr/lib/x86_64-linux-gnu/libX11.so;/usr/lib/x86_64-linux-gnu/libXext.so
-- Looking for XOpenDisplay in /usr/lib/x86_64-linux-gnu/libX11.so;/usr/lib/x86_64-linux-gnu/libXext.so - found
-- Looking for gethostbyname
-- Looking for gethostbyname - found
-- Looking for connect
-- Looking for connect - found
-- Looking for remove
-- Looking for remove - found
-- Looking for shmat
-- Looking for shmat - found
-- Building with Qt5.15.3
-- Found PkgConfig: /usr/bin/pkg-config (found version "0.29.2") 
-- Checking for module 'muparser'
--   Found muparser, version 2.2.6
-- Found Perl: /usr/bin/perl (found version "5.34.0") 
-- Configuring done
-- Generating done
CMake Warning:
  Manually-specified variables were not used by the project:                                                                                                 
                                                                                                                                                             
    LIB_SUFFIX                                                                                                                                               
                                                                                                                                                             
                                                                                                                                                             
-- Build files have been written to: /home/apaz/lxqt/lxqt-runner/build
```

#### Output 2

```
[  1%] Automatic MOC and UIC for target lxqt-runner
[  1%] Built target lxqt-runner_autogen
[  3%] Generating lxqt-runner_zh_TW.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_zh_TW.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[  5%] Generating lxqt-runner_ar.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_ar.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[  7%] Generating lxqt-runner_arn.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_arn.qm'...
    Generated 0 translation(s) (0 finished and 0 unfinished)
    Ignored 21 untranslated source text(s)
[  8%] Generating lxqt-runner_ast.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_ast.qm'...
    Generated 0 translation(s) (0 finished and 0 unfinished)
    Ignored 21 untranslated source text(s)
[ 10%] Generating lxqt-runner_bg.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_bg.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 12%] Generating lxqt-runner_ca.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_ca.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 14%] Generating lxqt-runner_cs.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_cs.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 15%] Generating lxqt-runner_cy.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_cy.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 17%] Generating lxqt-runner_da.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_da.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 19%] Generating lxqt-runner_de.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_de.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 21%] Generating lxqt-runner_el.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_el.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 22%] Generating lxqt-runner_eo.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_eo.qm'...
    Generated 6 translation(s) (6 finished and 0 unfinished)
    Ignored 15 untranslated source text(s)
[ 24%] Generating lxqt-runner_es.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_es.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 26%] Generating lxqt-runner_es_VE.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_es_VE.qm'...
    Generated 6 translation(s) (6 finished and 0 unfinished)
    Ignored 15 untranslated source text(s)
[ 28%] Generating lxqt-runner_et.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_et.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 29%] Generating lxqt-runner_eu.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_eu.qm'...
    Generated 16 translation(s) (12 finished and 4 unfinished)
    Ignored 5 untranslated source text(s)
[ 31%] Generating lxqt-runner_fi.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_fi.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 33%] Generating lxqt-runner_fr.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_fr.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 35%] Generating lxqt-runner_gl.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_gl.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 36%] Generating lxqt-runner_he.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_he.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 38%] Generating lxqt-runner_hr.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_hr.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 40%] Generating lxqt-runner_hu.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_hu.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 42%] Generating lxqt-runner_ia.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_ia.qm'...
    Generated 0 translation(s) (0 finished and 0 unfinished)
    Ignored 21 untranslated source text(s)
[ 43%] Generating lxqt-runner_id.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_id.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 45%] Generating lxqt-runner_it.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_it.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 47%] Generating lxqt-runner_ja.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_ja.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 49%] Generating lxqt-runner_ko.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_ko.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 50%] Generating lxqt-runner_lt.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_lt.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 52%] Generating lxqt-runner_nb_NO.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_nb_NO.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 54%] Generating lxqt-runner_nl.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_nl.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 56%] Generating lxqt-runner_pl.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_pl.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 57%] Generating lxqt-runner_pt.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_pt.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 59%] Generating lxqt-runner_pt_BR.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_pt_BR.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 61%] Generating lxqt-runner_ro_RO.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_ro_RO.qm'...
    Generated 19 translation(s) (19 finished and 0 unfinished)
    Ignored 2 untranslated source text(s)
[ 63%] Generating lxqt-runner_ru.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_ru.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 64%] Generating lxqt-runner_si.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_si.qm'...
    Generated 0 translation(s) (0 finished and 0 unfinished)
    Ignored 21 untranslated source text(s)
[ 66%] Generating lxqt-runner_sk.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_sk.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 68%] Generating lxqt-runner_sl.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_sl.qm'...
    Generated 20 translation(s) (20 finished and 0 unfinished)
    Ignored 1 untranslated source text(s)
[ 70%] Generating lxqt-runner_sr@latin.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_sr@latin.qm'...
    Generated 0 translation(s) (0 finished and 0 unfinished)
    Ignored 21 untranslated source text(s)
[ 71%] Generating lxqt-runner_sr_BA.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_sr_BA.qm'...
    Generated 6 translation(s) (6 finished and 0 unfinished)
    Ignored 15 untranslated source text(s)
[ 73%] Generating lxqt-runner_sr_RS.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_sr_RS.qm'...
    Generated 6 translation(s) (6 finished and 0 unfinished)
    Ignored 15 untranslated source text(s)
[ 75%] Generating lxqt-runner_th_TH.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_th_TH.qm'...
    Generated 6 translation(s) (6 finished and 0 unfinished)
    Ignored 15 untranslated source text(s)
[ 77%] Generating lxqt-runner_tr.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_tr.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 78%] Generating lxqt-runner_uk.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_uk.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 80%] Generating lxqt-runner_zh_CN.qm
Updating '/home/apaz/lxqt/lxqt-runner/build/lxqt-runner_zh_CN.qm'...
    Generated 21 translation(s) (21 finished and 0 unfinished)
[ 82%] Building CXX object CMakeFiles/lxqt-runner.dir/lxqt-runner_autogen/mocs_compilation.cpp.o
[ 84%] Building CXX object CMakeFiles/lxqt-runner.dir/main.cpp.o
[ 85%] Building CXX object CMakeFiles/lxqt-runner.dir/dialog.cpp.o
[ 87%] Building CXX object CMakeFiles/lxqt-runner.dir/commanditemmodel.cpp.o
[ 89%] Building CXX object CMakeFiles/lxqt-runner.dir/widgets.cpp.o
[ 91%] Building CXX object CMakeFiles/lxqt-runner.dir/providers.cpp.o
[ 92%] Building CXX object CMakeFiles/lxqt-runner.dir/yamlparser.cpp.o
[ 94%] Building CXX object CMakeFiles/lxqt-runner.dir/configuredialog/configuredialog.cpp.o
[ 96%] Building CXX object CMakeFiles/lxqt-runner.dir/LXQtAppTranslationLoader.cpp.o
[ 98%] Linking CXX executable lxqt-runner
[ 98%] Built target lxqt-runner
[100%] Generating lxqt-runner.desktop
[100%] Built target lxq_runner_autostart_desktop_files
```

#### Output 4

```
[  1%] Automatic MOC and UIC for target lxqt-runner
[  1%] Built target lxqt-runner_autogen
Consolidate compiler generated dependencies of target lxqt-runner
[ 98%] Built target lxqt-runner
[100%] Built target lxq_runner_autostart_desktop_files
Install the project...
-- Install configuration: "debug"
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_ar.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_arn.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_ast.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_bg.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_ca.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_cs.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_cy.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_da.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_de.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_el.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_eo.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_es.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_es_VE.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_et.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_eu.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_fi.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_fr.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_gl.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_he.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_hr.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_hu.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_ia.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_id.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_it.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_ja.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_ko.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_lt.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_nb_NO.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_nl.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_pl.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_pt.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_pt_BR.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_ro_RO.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_ru.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_si.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_sk.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_sl.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_sr@latin.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_sr_BA.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_sr_RS.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_th_TH.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_tr.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_uk.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_zh_CN.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/lxqt/translations/lxqt-runner/lxqt-runner_zh_TW.qm
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/bin/lxqt-runner
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/usr/share/man/man1/lxqt-runner.1
-- Installing: /home/apaz/lxqt/lxqt-runner/build/install/etc/xdg/autostart/lxqt-runner.desktop
```
