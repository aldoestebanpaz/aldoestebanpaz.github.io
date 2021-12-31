---
title: "GNU Make, advanced tips and tricks"
tags: usage make gnu-make
---

# GNU Make, advanced tips and tricks

## Debugging

### Print the data base of predefined rules and variables

To print the data base of predefined rules and variables, use:

```
'make -p -f /dev/null'
```

### Command parameters for debugging

```
make -w -d -n -p
```

Where:

* '-w' or '--print-directory' print a message containing the working directory both before and after executing the makefile.

* '-d' or '--debug=a' prints debugging information in addition to normal processing. The debugging information says which files are being considered for remaking, which file-times are being compared and with what results, which files actually need to be remade, which implicit rules are considered and which are applied—everything interesting about how make decides what to do.

* '-n',
'--just-print',
'--dry-run' or
'--recon' causes make to print the recipes that are needed to make the targets up to date, but not actually execute them.

* '-p' or '--print-data-base' prints the data base (rules and variable values) that results from reading the makefiles; then execute as usual or as otherwise specified. The data base output contains file name and line number information for recipe and variable definitions, so it can be a useful debugging tool in complex environments.

### Debugging with strace

```
strace -o ./mytrace.log -s 2048 -f -e trace=/exec,file \
make -w -d -n > make.debug
```

### Debugging the linux kernel build system

```
make -w -d -n V=1 | tee make.debug
```

or

```
make -w -d -n V=1 TARGET | tee make.debug
```

e.g. `make -w -d -n V=1 pdfdocs | tee make.debug`

## Code snippets

All the code snippets here were extracted from the Linux kernel 5.15 build system (more specificaly from [the latest commit](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=d2f38a3c6507b2520101f9a3807ed98f1bdc545a) that was on the master branch when I cloned the repo).

### Standarize and normalize common environment variables

Definning a custom value of those global environment variables is considered a
good practice, specially for big projects. The following is an example of
unexport and export of common environment variables.

```
# Avoid funny character set dependencies
unexport LC_ALL
LC_COLLATE=C
LC_NUMERIC=C
export LC_COLLATE LC_NUMERIC

# Avoid interference with shell env settings
unexport GREP_OPTIONS
```

### Adding a 'help' target

For big projects could be a good a good idea adding a 'help' target to show the typical targets, that way calling 'make help' you can remember some common commands.

```
PHONY := __all
__all:

# What is the source directory?
this-makefile := $(lastword $(MAKEFILE_LIST))
abs_srctree := $(realpath $(dir $(this-makefile)))
srctree := $(abs_srctree)


SRCARCH := $(shell uname -m | sed -e s/i.86/x86/ -e s/x86_64/x86/ \
				  -e s/sun4u/sparc64/ \
				  -e s/arm.*/arm/ -e s/sa110/arm/ \
				  -e s/s390x/s390/ \
				  -e s/ppc.*/powerpc/ -e s/mips.*/mips/ \
				  -e s/sh[234].*/sh/ -e s/aarch64.*/arm64/ \
				  -e s/riscv.*/riscv/)

INSTALLKERNEL  := installkernel
export INSTALLKERNEL


include $(srctree)/arch/$(SRCARCH)/Makefile


#
# INSTALL_PATH specifies where to place the updated kernel and system map
# images. Default is /boot, but you can set it to other values
export	INSTALL_PATH ?= /boot


PHONY += help
help:
	@echo  'Cleaning targets:'
	@echo  '  clean       - Remove most generated files but keep the config and'
	@echo  '                enough build support to build external modules'
	@echo  '  mrproper	  - Remove all generated files + config + various backup files'
	@echo  '  distclean	  - mrproper + remove editor backup and patch files'
	@echo  ''
	@echo  'Configuration targets:'
	@$(MAKE) -f $(srctree)/scripts/kconfig/Makefile help
	@echo  ''
	@echo  'Other generic targets:'
	@echo  '  all         - Build all targets marked with [*]'
	@echo  '* vmlinux     - Build the bare kernel'
	@echo  '* modules     - Build all modules'
    ...
    @echo  ''
	@echo  'Architecture specific targets ($(SRCARCH)):'
	@$(if $(archhelp),$(archhelp),\
		echo '  No architecture specific help defined for $(SRCARCH)')
	@echo  ''
    ...
    @echo  ''
	@echo  'Execute "make" or "make all" to build all targets marked with [*] '
	@echo  'For further info see the ./README file'



PHONY += FORCE
FORCE:

# Declare the contents of the PHONY variable as phony.
.PHONY: $(PHONY)
```

and the content of the '$(srctree)/arch/$(SRCARCH)/Makefile' file could be:

```
define archhelp
  echo  '* bzImage     - Compressed kernel image (arch/x86/boot/bzImage)'
  echo  '  install     - Install kernel using (your) ~/bin/$(INSTALLKERNEL) or'
  echo  '                (distribution) /sbin/$(INSTALLKERNEL) or install to '
  echo  '                $$(INSTALL_PATH) and run lilo'
  echo  ''
  echo  '  fdimage     - Create 1.4MB boot floppy image (arch/x86/boot/fdimage)'
  ...
  echo  ''
  echo  '  kvm_guest.config	- Enable Kconfig items for running this kernel as a KVM guest'
  echo  '  xen.config  - Enable Kconfig items for running this kernel as a Xen guest'

endef
```

### Make options inside Makefile

GNU Make has an special variable called MAKEFLAGS. This variable is set up
automatically by make to contain the flag letters that make received. Thus, if
you do 'make -ks' then MAKEFLAGS gets the value 'ks'.

Two marked functionalities inherites from this variable:

(1) Hardcode configurations inside the Makefile

You could simply put a value for MAKEFLAGS in your environment, or also set
MAKEFLAGS in a makefile, to specify additional flags that should also be in
effect for that makefile.

When make interprets the value of MAKEFLAGS (either from the environment or from
a makefile), it first prepends a hyphen if the value does not already begin with
one. Then it chops the value into words separated by blanks, and parses these
words as if they were options given on the command line (except that ‘-C’, ‘-f’,
‘-h’, ‘-o’, ‘-W’, and their long-named versions are ignored; and there is no
error for an invalid option).

(2) Passing options to a sub-make automatically

Like before, sub-make gets a value for MAKEFLAGS in its environment. In
response, it takes the flags from that value and processes them as if they had
been given as arguments.

Likewise variables defined on the command line are passed to the sub-make
through MAKEFLAGS. Words in the value of MAKEFLAGS that contain '=', make treats
as variable definitions just as if they appeared on the command line.

Additionally, special variable MAKEFLAGS is always exported (unless you unexport
it). MAKEFILES is exported if you set it to anything.

The options ‘-C’, ‘-f’, ‘-o’, and ‘-W’ are not put into MAKEFLAGS; these options
are not passed down.

```
# Do not use make's built-in rules and variables
# (this increases performance and avoids hard-to-debug behaviour)
MAKEFLAGS += -rR
```

### Configurable verbosity

(1) Set the @ prefix, that supress echoing of recipe lines, configuring a variable

Normally make prints each line of the recipe before it is executed. We call this
echoing because it gives the appearance that you are typing the lines yourself.

We could prefix commands with a variable $(Q); like in the following code
snippet, for commands that shall be hidden in non-verbose mode. In the following
example we can use a command like 'make V=1' to see the full commands, or just
'make' for less verbose as default.

(2) Chosing oter forms of output

By making echo '$($(quiet)$(cmd))', we have the possibility to set $(quiet) to
choose other forms of output instead, e.g.
```
quiet_cmd_cc_o_c = Compiling $(RELDIR)/$@
cmd_cc_o_c       = $(CC) $(c_flags) -c -o $@ $<
```
- If $(quiet) is empty, the whole command will be printed.
- If it is set to "quiet_", only the short version will be printed.
- If it is set to "silent_", nothing will be printed at all, since the variable
  '$(silent_cmd_cc_o_c)' doesn't exist.


```
ifeq ("$(origin V)", "command line")
  BUILD_VERBOSE = $(V)
endif
ifndef KBUILD_VERBOSE
  BUILD_VERBOSE = 0
endif

ifeq ($(BUILD_VERBOSE),1)
  quiet =
  Q =
else
  quiet=quiet_
  Q = @
endif

# If the user is running make -s (silent mode), suppress echoing of
# commands

ifneq ($(findstring s,$(filter-out --%,$(MAKEFLAGS))),)
  quiet=silent_
  BUILD_VERBOSE = 0
endif

export quiet Q BUILD_VERBOSE

# Clear a bunch of variables before executing the submake
ifeq ($(quiet),silent_)
tools_silent=s
endif

# Convenient variables
squote  := '

# Escape single quote for use in echo statements
escsq = $(subst $(squote),'\$(squote)',$1)

# echo command.
# Short version is used, if $(quiet) equals `quiet_', otherwise full one.
echo-cmd = $(if $($(quiet)cmd_$(1)),\
	echo '  $(call escsq,$($(quiet)cmd_$(1)))';)

# sink stdout for 'make -s'
       redirect :=
 quiet_redirect :=
silent_redirect := exec >/dev/null;

# printing commands
cmd = @set -e; $(echo-cmd) $($(quiet)redirect) $(cmd_$(1))


# sample recipe using Q and tools_silent variable
tools: FORCE
	$(Q)mkdir -p ./tools
	$(Q)$(MAKE)  MAKEFLAGS="$(tools_silent)"

CLEAN_FILES += include/ksym vmlinux.symvers modules-only.symvers \
	       modules.builtin modules.builtin.modinfo modules.nsdeps \
	       compile_commands.json .thinlto-cache

clean: rm-files := $(CLEAN_FILES)

clean:
	$(call cmd,rmfiles)

quiet_cmd_rmfiles = $(if $(wildcard $(rm-files)),CLEAN   $(wildcard $(rm-files)))
      cmd_rmfiles = rm -rf $(rm-files)

PHONY := tools clean
PHONY += FORCE
FORCE:

# Declare the contents of the PHONY variable as phony.  We keep that
# information in a variable so we can use it in if_changed and friends.
.PHONY: $(PHONY)
```

### Convert all targets to callable targets via command line to phony targets

This code snippet allows the creation of a Makefile that only have targets that
represents pure actions (phony target) without having to name them all in the
.PHONY variable.

This takes advantage of the GNU Make special variable MAKECMDGOALS that has the
list of goals you specified on the command line. If no goals were given on the
command line, this variable is empty.

```
PHONY := $(MAKECMDGOALS)

...

PHONY += FORCE
FORCE:

# Declare the contents of the PHONY variable as phony.
.PHONY: $(PHONY)
```

### Creating phony targets for internal use only

The following code snippet introduces a condition that hides phony targets
starting with '__' of being called explicatly from command line.

```
$(if $(filter __%, $(MAKECMDGOALS)), \
	$(error targets prefixed with '__' are only for internal use))

# __all will be our default target when none is given on the command line
# you cannot call it explicetly with "make __all" because of the condition above
PHONY := __all
__all:
    ...

PHONY += __another_hidden_target
__another_hidden_target:
    ...

PHONY += FORCE
FORCE:

# Declare the contents of the PHONY variable as phony.
.PHONY: $(PHONY)
```

### Recursive build

We can use the recursive invocation concept to calling the Makefile itself as it
were a sub-make file. This allows the separation of setting some variables and
checking some values in the first invocation and use those variables in the
second.

Also specifying '-C' in te second invocation allows you to set another value
to the special variable CURDIR which points to the working directory.

```
PHONY := __all
__all:



# First invocation block
ifneq ($(sub_make),1)


# PUT HERE ANY CHECKS ANY VARIABLES TO EXPORT TO SECOND INVOCATION


# Do not use make's built-in rules and variables
# (this increases performance and avoids hard-to-debug behaviour)
MAKEFLAGS += -rR


# What is the source directory?
this-makefile := $(lastword $(MAKEFILE_LIST))
abs_srctree := $(realpath $(dir $(this-makefile)))
srctree := $(abs_srctree)

ifneq ($(words $(subst :, ,$(abs_srctree))), 1)
$(error source directory cannot contain spaces or colons)
endif


# Do we want to change the working directory?
ifeq ("$(origin O)", "command line")
  OUTPUT_DIR := $(O)
endif

ifneq ($(OUTPUT_DIR),)
# Make's built-in functions such as $(abspath ...), $(realpath ...) cannot
# expand a shell special character '~'. We use a somewhat tedious way here.
abs_objtree := $(shell mkdir -p $(OUTPUT_DIR) && cd $(OUTPUT_DIR) && pwd)
$(if $(abs_objtree),, \
     $(error failed to create output directory "$(OUTPUT_DIR)"))

# $(realpath ...) resolves symlinks
abs_objtree := $(realpath $(abs_objtree))
else
abs_objtree := $(CURDIR)
endif # ifneq ($(OUTPUT_DIR),)


# CONDITIONS FOR SECOND INVOCATION
ifeq ($(abs_objtree),$(CURDIR))
# Suppress "Entering directory ..." unless we are changing the work directory.
MAKEFLAGS += --no-print-directory
else
need-sub-make := 1
endif

ifneq ($(abs_srctree),$(abs_objtree))
# Look for make include files relative to root of kernel src
# --included-dir is added for backward compatibility, but you should not rely on
# it. Please add $(srctree)/ prefix to include Makefiles in the source tree.
MAKEFLAGS += --include-dir=$(abs_srctree)
endif

ifneq ($(filter 3.%,$(MAKE_VERSION)),)
# 'MAKEFLAGS += -rR' does not immediately become effective for GNU Make 3.x
# We need to invoke sub-make to avoid implicit rules in the top Makefile.
need-sub-make := 1
# Cancel implicit rules for this Makefile.
$(this-makefile): ;
endif

# EXPORTS
# Set sub_make as 1 and export it to skip first invocation block
# in the second invocation.
# Also export any needed variable for second invocation.
export sub_make := 1
export srctree


# Do we need to make a second invocation?
ifeq ($(need-sub-make),1)

PHONY += $(MAKECMDGOALS) __sub-make

$(filter-out $(this-makefile), $(MAKECMDGOALS)) __all: __sub-make
	@:

# Invoke a second make in the output directory, passing relevant variables
__sub-make:
	$(MAKE) -C $(abs_objtree) -f $(abs_srctree)/Makefile $(MAKECMDGOALS)

endif # need-sub-make
endif # sub_make




# We process the rest of the Makefile if this is the final invocation of make
ifeq ($(need-sub-make),)
# Second invocation block
...
endif # need-sub-make




PHONY += FORCE
FORCE:

# Declare the contents of the PHONY variable as phony.
.PHONY: $(PHONY)
```
