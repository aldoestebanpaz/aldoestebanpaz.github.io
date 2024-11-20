# Cmake usage

- [Cmake usage](#cmake-usage)
  - [Build](#build)
  - [Build types](#build-types)
  - [Cmake packages](#cmake-packages)
    - [Module mode](#module-mode)
    - [Config mode](#config-mode)
      - [Config file search procedure](#config-file-search-procedure)
      - [Example](#example)
    - [Using pkg-config packages](#using-pkg-config-packages)
  - [Troubleshooting](#troubleshooting)
    - [How to find necessary packages to use with pkg\_](#how-to-find-necessary-packages-to-use-with-pkg_)
    - [How to find necessary packages to use with pkg\_check\_modules()](#how-to-find-necessary-packages-to-use-with-pkg_check_modules)
    - [Stop a CMake run in an specific point without an error](#stop-a-cmake-run-in-an-specific-point-without-an-error)
      - [Using return()](#using-return)
      - [Kill the process abruptly](#kill-the-process-abruptly)
    - [List variables using cmake command](#list-variables-using-cmake-command)
    - [List all calculated variables](#list-all-calculated-variables)
    - [Printing variables with cmake code](#printing-variables-with-cmake-code)
    - [Tracing a build](#tracing-a-build)
      - [Trace lines in your CMakeLists.txt](#trace-lines-in-your-cmakeliststxt)
      - [Complete trace](#complete-trace)
      - [Trace and expanded variables](#trace-and-expanded-variables)
      - [Trace and debug information](#trace-and-debug-information)
      - [Get find\_\* call lookups](#get-find_-call-lookups)
  - [References](#references)
    - [Cmake variables](#cmake-variables)
    - [Common cmake commands](#common-cmake-commands)
    - [Common modules](#common-modules)
    - [Guides and tutorials](#guides-and-tutorials)

## Build

1. Configure the project:

```sh
mkdir -v build
cd       build

cmake -DCMAKE_BUILD_TYPE=Release \
      ..
```

2. Compile and link:

```sh
make
```

3. Install:

```sh
# as root:
make install
```

## Build types

You can run CMake with "CMAKE_BUILD_TYPE=Debug" for a build with full debugging, or "RelWithDebInfo" for a release build with some extra debug info. You can also use "Release" for an optimized release build, or "MinSizeRel" for a minimum size release (which I’ve never used).

## Cmake packages

The `find_package` command finds and loads settings from an external project. When the package is found package-specific information is provided through variables and Imported Targets documented by the package itself.

Reduced signature:

```
find_package(<package> [version] [EXACT] [QUIET] [MODULE]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS components...]
             [NO_POLICY_SCOPE])
```

Complete config mode command signature:

```
find_package(<package> [version] [EXACT] [QUIET]
             [REQUIRED] [[COMPONENTS] [components...]]
             [CONFIG|NO_MODULE]
             [NO_POLICY_SCOPE]
             [NAMES name1 [name2 ...]]
             [CONFIGS config1 [config2 ...]]
             [HINTS path1 [path2 ... ]]
             [PATHS path1 [path2 ... ]]
             [PATH_SUFFIXES suffix1 [suffix2 ...]]
             [NO_DEFAULT_PATH]
             [NO_CMAKE_PATH]
             [NO_CMAKE_ENVIRONMENT_PATH]
             [NO_SYSTEM_ENVIRONMENT_PATH]
             [NO_CMAKE_PACKAGE_REGISTRY]
             [NO_CMAKE_BUILDS_PATH] # Deprecated; does nothing.
             [NO_CMAKE_SYSTEM_PATH]
             [NO_CMAKE_SYSTEM_PACKAGE_REGISTRY]
             [CMAKE_FIND_ROOT_PATH_BOTH |
              ONLY_CMAKE_FIND_ROOT_PATH |
              NO_CMAKE_FIND_ROOT_PATH])
```

`<package>_FOUND` will be set to indicate whether the package was found.

The command has two modes by which it searches for packages:

- Module mode - packages provided for this mode are known as Find-module Packages.
- Config mode - packages provided for this mode are known as Config-file Packages.

More info: https://cmake.org/cmake/help/v3.10/manual/cmake-packages.7.html

### Module mode

This mode is available when the command is invoked with the reduced signature. CMake searches for a file called `Find<package>.cmake` in the `CMAKE_MODULE_PATH` followed by the CMake installation. If the file is found, it is read and processed by CMake. It is responsible for finding the package, checking the version, and producing any needed messages. Many find-modules provide limited or no support for versioning; check the module documentation.

### Config mode

If no module is found, and the MODULE option is not given, the command proceeds to Config mode. Config mode attempts to locate a configuration file provided by the package to be found.

A cache entry called `<package>_DIR` is created to hold the directory containing the file.

By default the command searches for a package with the name `<package>`. If the `NAMES` option is given the names following it are used instead of `<package>`. The command searches for a file called `<name>Config.cmake` or `<lower-case-name>-config.cmake` for each name specified. A replacement set of possible configuration file names may be given using the `CONFIGS` option.

Once found, the configuration file is read and processed by CMake. Since the file is provided by the package it already knows the location of package contents. The full path to the configuration file is stored in the cmake variable `<package>_CONFIG`.

#### Config file search procedure

CMake constructs a set of possible installation prefixes for the package. Under each prefix several directories are searched for a configuration file. The table below show the directories searched. Each entry is meant for installation trees following Windows (W), UNIX (U), or Apple (A) conventions:

```
<prefix>/                                                       (W)
<prefix>/(cmake|CMake)/                                         (W)
<prefix>/<name>*/                                               (W)
<prefix>/<name>*/(cmake|CMake)/                                 (W)
<prefix>/(lib/<arch>|lib|share)/cmake/<name>*/                  (U)
<prefix>/(lib/<arch>|lib|share)/<name>*/                        (U)
<prefix>/(lib/<arch>|lib|share)/<name>*/(cmake|CMake)/          (U)
<prefix>/<name>*/(lib/<arch>|lib|share)/cmake/<name>*/          (W/U)
<prefix>/<name>*/(lib/<arch>|lib|share)/<name>*/                (W/U)
<prefix>/<name>*/(lib/<arch>|lib|share)/<name>*/(cmake|CMake)/  (W/U)
```

In all cases the `<name>` is treated as case-insensitive and corresponds to any of the names specified (`<package>` or names given by `NAMES`). Paths with `lib/<arch>` are enabled if the `CMAKE_LIBRARY_ARCHITECTURE` variable is set. If `PATH_SUFFIXES` is specified the suffixes are appended to each (W) or (U) directory entry one-by-one.

#### Example

The [lxqt-about](https://github.com/lxqt/lxqt-about) package has the following commands in his own CMakeLists.txt file:

```cmake
set(LXQT_MINIMUM_VERSION "1.0.0")

find_package(lxqt ${LXQT_MINIMUM_VERSION} REQUIRED)

include(LXQtPreventInSourceBuilds)
include(LXQtCompilerSettings NO_POLICY_SCOPE)
...
include(LXQtTranslate)
```

In my Ubuntu system the following are the cmake config-file packages (mentioned in the `find_package` command):

- `lxqt` in `/usr/share/cmake/lxqt/lxqt-config.cmake`, provided by [liblxqt0-dev](https://github.com/lxqt/liblxqt).
- `lxqt-build-tools` in `/usr/share/cmake/lxqt-build-tools/lxqt-build-tools-config.cmake`, provided by [lxqt-build-tools](https://github.com/lxqt/lxqt-build-tools).

The `lxqt-build-tools` package dependency does not appears above but in `lxqt-config.cmake` instead, among other dependencies, with the following command:

```
find_dependency(lxqt-build-tools 0.8.0)
```

`find_dependency` is a macro provided by the [CMakeFindDependencyMacro](https://cmake.org/cmake/help/latest/module/CMakeFindDependencyMacro.html) module that basically wraps a `find_package()` call.

`lxqt-build-tools` is the config-file package that takes care of adding the modules to `CMAKE_MODULE_PATH`, making these available to any CMakeLists.txt file calling `find_package(lxqt ${LXQT_MINIMUM_VERSION} REQUIRED)`:

```
set(LXQT_CMAKE_MODULES_DIR "${PACKAGE_PREFIX_DIR}/share/cmake/lxqt-build-tools/modules/")
set(LXQT_CMAKE_FIND_MODULES_DIR "${PACKAGE_PREFIX_DIR}/share/cmake/lxqt-build-tools/find-modules/")

list(APPEND CMAKE_MODULE_PATH
    "${LXQT_CMAKE_MODULES_DIR}"
    "${LXQT_CMAKE_FIND_MODULES_DIR}"
)
```

Finally the modules mentioned in the `include` command are provided by the [lxqt-build-tools](https://github.com/lxqt/lxqt-build-tools) package too:

- `LXQtPreventInSourceBuilds` in `/usr/share/cmake/lxqt-build-tools/modules/LXQtPreventInSourceBuilds.cmake`.
- `LXQtCompilerSettings` in `/usr/share/cmake/lxqt-build-tools/modules/LXQtCompilerSettings.cmake`.
- `LXQtTranslate` in `/usr/share/cmake/lxqt-build-tools/modules/LXQtTranslate.cmake`.

### Using pkg-config packages

TODO

## Troubleshooting

### How to find necessary packages to use with pkg_

Example, the following code searches cmake config files for Qt5 development:

```cmake
# ...

set(QT_MAJOR_VERSION 5)

# ...

find_package(Qt${QT_MAJOR_VERSION} 5.15.0 CONFIG REQUIRED Core DBus Gui Qml Quick LinguistTools Test QuickTest)
```

To find the necessary package I can use the following commands on apt systems:

```sh
apt-file search --regexp '.*Qt5Core.*\.cmake'
#   ...
#   qtbase5-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5Core/Qt5CoreConfig.cmake
#   ...
#   qtbase5-gles-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5Core/Qt5CoreConfig.cmake
#   ...

apt-file search --regexp '.*Qt5DBus.*\.cmake'
#   ...
#   qtbase5-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5DBus/Qt5DBusConfig.cmake
#   ...
#   qtbase5-gles-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5DBus/Qt5DBusConfig.cmake
#   ...

apt-file search --regexp '.*Qt5QuickTest.*\.cmake'
#   ...
#   qtdeclarative5-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5QuickTest/Qt5QuickTestConfig.cmake
#   ...

apt-file search Qt5GuiConfig.cmake
#   qtbase5-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5Gui/Qt5GuiConfig.cmake
#   qtbase5-gles-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5Gui/Qt5GuiConfig.cmake

apt-file search Qt5QmlConfig.cmake
#   qtdeclarative5-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5Qml/Qt5QmlConfig.cmake

apt-file search Qt5QuickConfig.cmake
#   qtdeclarative5-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5Quick/Qt5QuickConfig.cmake

apt-file search Qt5LinguistToolsConfig.cmake
#   qttools5-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5LinguistTools/Qt5LinguistToolsConfig.cmake

apt-file search Qt5TestConfig.cmake
#   qtbase5-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5Test/Qt5TestConfig.cmake
#   qtbase5-gles-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5Test/Qt5TestConfig.cmake

apt-file search Qt5QuickTestConfig.cmake
#   qtdeclarative5-dev: /usr/lib/x86_64-linux-gnu/cmake/Qt5QuickTest/Qt5QuickTestConfig.cmake
```

### How to find necessary packages to use with pkg_check_modules()

Example, the following code enables pkg-config (`FindPkgConfig.cmake` is already included with cmake) and parses `xau.pc` and `xcb.pc`:

```cmake
find_package(PkgConfig)
pkg_check_modules(LIBXAU REQUIRED "xau")
pkg_check_modules(PKG_XCB xcb)
```

To find the necessary package I can use the following commands on apt systems:

```sh
apt-file search xau.pc
#   libxau-dev: /usr/lib/x86_64-linux-gnu/pkgconfig/xau.pc
apt-file search xcb.pc
#   ...
#   libxcb1-dev: /usr/lib/x86_64-linux-gnu/pkgconfig/xcb.pc
```

### Stop a CMake run in an specific point without an error

#### Using return()

If your CMakeLists.txt hierarchy is fairly shallow, you can call [return()](https://cmake.org/cmake/help/latest/command/return.html) once or twice (depending on if you called add_subdirectory() or if you're in a macro/function) to return to the calling file or function and exit CMake processing.

Example:

```cmake
# ...

message(STATUS "BREAKPOINT ---------------------------------------------")
return()

# ...
```

#### Kill the process abruptly

If your project hierarchy is more complex, and you want to exit your CMakes from an arbitrary location, there is no native CMake command to support that. However, you could roll your own (admittedly, a bit scary) solution to terminate cmake.

If you include this in your top-level CMake file, you can call this from anywhere in your CMakes to exit silently.

WARNING: This will terminate ALL currently running CMake processes, not just your current process. Because you are forcing the process the terminate, your CMake files/cache may be left in a weird state.

```cmake
function(exit_cmake)
  if(UNIX)
    set(KILL_COMMAND "killall")
    execute_process(COMMAND ${KILL_COMMAND} -9 cmake
      WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
    )
  else()
    set(KILL_COMMAND "taskkill")
    execute_process(COMMAND ${KILL_COMMAND} /IM cmake.exe /F
      WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR}
    )
  endif()
endfunction()
```

### List variables using cmake command

The command `cmake -L[A][H]` lists non-advanced cached variables.

The above command lists all the variables from the CMake CACHE that are not marked as INTERNAL or ADVANCED. This will effectively display current CMake settings, which can then be changed with -D option. Changing some of the variables may result in more variables being created. If A is specified, then it will display also advanced variables. If H is specified, it will also display help for each variable.

```sh
cd build
# Build with: cmake ..
cmake -LAH .
```

### List all calculated variables

```sh
cd build
# Build with: cmake ..
less CMakeCache.txt
```

### Printing variables with cmake code

**Option 1: the time honored method of print statements looks like this**

```cmake
message(STATUS "MY_VARIABLE=${MY_VARIABLE}")
```

**Option 2: a built in module makes this even easier**

```cmake
include(CMakePrintHelpers)
cmake_print_variables(MY_VARIABLE)
```

**Option 3: printing properties**

Instead of getting the properties one by one of each target (or other item with properties, such as SOURCES, DIRECTORIES, TESTS, or CACHE_ENTRIES - global properties seem to be missing for some reason), you can simply list them and get them printed directly.

NOTE: You can't actually access SOURCES, since it conflictes with the SOURCES keyword in the function.

```cmake
cmake_print_properties(
    TARGETS my_target
    PROPERTIES POSITION_INDEPENDENT_CODE
)
```

### Tracing a build

#### Trace lines in your CMakeLists.txt

With `--trace-source="filename"` every line run in the file that you give will be echoed to the screen when it is run, letting you follow exactly what is happening.

```cmake
cd <PROJECTDIR>

mkdir build

cmake \
  -S . \
  -B build \
  --trace-source=CMakeLists.txt
```

#### Complete trace

With `--trace` every line run, not just those from your CMakeLists.txt but all included files too, will be echoed to the screen.

```cmake
cd <PROJECTDIR>

mkdir build

cmake \
  -S . \
  -B build \
  --trace
```

#### Trace and expanded variables

Add `--trace-expand` to print expanded variables.

```cmake
cd <PROJECTDIR>

mkdir build

cmake \
  -S . \
  -B build \
  --trace \ # or --trace-source=CMakeLists.txt
  --trace-expand
```

#### Trace and debug information

Add `--debug-output` to print debug information.

```cmake
cd <PROJECTDIR>

mkdir build

cmake \
  -S . \
  -B build \
  --trace \ # or --trace-source=CMakeLists.txt
  # --trace-expand \ # trace with expanded variables
  --debug-output
```

#### Get find_* call lookups

You can print extra find_* call information during the cmake run to standard error by adding `--debug-find` (CMake 3.17+).

Alternatively, [CMAKE_FIND_DEBUG_MODE](https://cmake.org/cmake/help/latest/variable/CMAKE_FIND_DEBUG_MODE.html) can be set around sections of your CMakeLists.txt to limit debug printing to a specific region.

## References

### Cmake variables

- [All](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html) - This page documents variables that are provided by CMake or have meaning to CMake when set by project code.

### Common cmake commands
- [cmake_minimum_required](https://cmake.org/cmake/help/latest/command/cmake_minimum_required.html).
- [project](https://cmake.org/cmake/help/latest/command/project.html).
- [set](https://cmake.org/cmake/help/latest/command/set.html).
- [option](https://cmake.org/cmake/help/latest/command/option.html).
- [find_package](https://cmake.org/cmake/help/latest/command/find_package.html).
- [include](https://cmake.org/cmake/help/latest/command/include.html).
- [add_definitions](https://cmake.org/cmake/help/latest/command/add_definitions.html).
- [add_executable](https://cmake.org/cmake/help/latest/command/add_executable.html).
- [target_link_libraries](https://cmake.org/cmake/help/latest/command/target_link_libraries.html).
- [install](https://cmake.org/cmake/help/latest/command/install.html).

### Common modules

- [GNUInstallDirs](https://cmake.org/cmake/help/latest/module/GNUInstallDirs.html).

### Guides and tutorials

- [HSF CMake tutorial - More Modern CMake](https://hsf-training.github.io/hsf-training-cmake-webpage/).
- [Modern CMake](https://cliutils.gitlab.io/modern-cmake/).