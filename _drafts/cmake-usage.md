# Cmake usage

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
