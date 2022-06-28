# Managing patches using 'quilt'

Quilt manages a series of patches by keeping track of the changes each of them makes. They are logically organized as a stack, and you can apply, un-apply, update them easily by traveling into the stack (push/pop).

Quilt is good for managing additional patches applied to a package received as a tarball or maintained in another version control system.

The "stack of patches" is maintained in a dedicated directory ('patches/' by default, but in Debian packages we override this value to 'debian/patches'). This directory contains the patch files and a 'series' file that gives an ordered list of patches to apply.

When quilt is used, it also maintains some internal files in a directory of its own (it's named '.pc/'). This directory is used to know what patches are currently applied ('.pc/applied-patches') and to keep backup copies of files modified by the various patches.

See `man quilt` for details.

## Basic example

### (1) Move into the workspace directory

```sh
mkdir quilt-example

cd $_
tee -a example-file-1 > /dev/null <<EOT
line 1
line 2
line 3
EOT
```

### (2) Starting with a new patch

`quilt new name-of-my-patch.diff` tells quilt to insert a new empty patch after the current topmost patch. In this operation quilt does almost nothing except updating the 'series' file and recording the fact that the new patch is applied (even if it's still empty).

NOTE: Quilt will always add the new patch just after the patch which is currently on top. So if you want to add the patch at the end of the series, you need to run `quilt push -a` first.

```sh
quilt new example-patch-1.diff
# Patch patches/example-patch-1.diff is now on top

ll
# .  ..  example-file-1  patches  .pc
```

### (3) Making and reviewing the changes

#### Alternative 1 - Using 'quilt add'

To add changes in a patch, quilt needs to be informed of the files you are trying to modify. You do this with `quilt add file-to-modify`. At this point quilt will make a backup copy of that file so that it can generate the final patch when you're done with your changes.

Now after you've executed 'quilt new ...' to start a new patch and 'quilt add ...' for each file that will be included in the patch, you can modify these files.

```sh
quilt add example-file-1
# File example-file-1 added to patch patches/example-patch-1.diff

vi $_

quilt diff
# Index: quilt-example/example-file-1
# ===================================================================
# --- quilt-example.orig/example-file-1
# +++ quilt-example/example-file-1
# @@ -1,3 +1,4 @@
#  line 1
# -line 2
# +edit 2
#  line 3
# +new line 4
```

#### Alternative 2 - Using 'quilt edit'

It's quite common to forget to execute 'quilt add ...' and to be unable to generate the patch afterward. That's why it is recommended to use `quilt edit file-to-modify` which is a shorthand for doing 'quilt add' and then opening the file in your favorite text editor.

```sh
quilt edit example-file-1
# File example-file-1 is already in patch patches/example-patch-1.diff

quilt diff
# Index: quilt-example/example-file-1
# ===================================================================
# --- quilt-example.orig/example-file-1
# +++ quilt-example/example-file-1
# @@ -1,3 +1,4 @@
#  line 1
# -line 2
# +edit 2
#  line 3
# +new line 4
```

### (4) Saving the changes into the patch

```sh
quilt refresh
# Refreshed patch patches/example-patch-1.diff
```

Now the changes are stored into the patch file 'example-patch-1.diff' in 'patches/', and is the one currently applied.

### (5) Adding more patches

```sh
quilt new example-patch-2.diff
# Patch patches/example-patch-2.diff is now on top


quilt add example-file-2
# File example-file-2 added to patch patches/example-patch-2.diff
echo "New file created by example-patch-2.diff" > example-file-2


quilt add example-file-1
# File example-file-1 added to patch patches/example-patch-2.diff
echo "Line created by example-patch-2.diff" >> example-file-1


quilt diff
# Index: quilt-example/example-file-1
# ===================================================================
# --- quilt-example.orig/example-file-1
# +++ quilt-example/example-file-1
# @@ -2,3 +2,4 @@ line 1
#  edit 2
#  line 3
#  new line 4
# +Line created by example-patch-2.diff
# Index: quilt-example/example-file-2
# ===================================================================
# --- /dev/null
# +++ quilt-example/example-file-2
# @@ -0,0 +1 @@
# +New file created by example-patch-2.diff


quilt refresh
# Refreshed patch patches/example-patch-2.diff
```

### (6) Navigating throw the stack of patches

We have the following commands:
- `quilt series` - Print the names of all patches in the series file.
- `quilt applied [patch]` - Print a list of applied patches, or all patches up to and including the specified patch in the file series.
- `quilt unapplied [patch]` - Print a list of patches that are not applied, or all patches that follow the specified patch in the series file.
- `quilt top` - Print the name of the topmost patch on the current stack of applied patches.
- `quilt previous [patch]` - Print the name of the previous patch before the specified or topmost patch in the series file.
- `quilt next [patch]` - Print the name of the next patch after the specified or topmost patch in the series file.
- `quilt graph [--lines[=num]] [patch]` - Generate a dot (see `man dot`) directed graph showing the dependencies between applied patches (a patch depends on another patch if both touch the same file or, with the --lines option, if their modifications overlap).
- `quilt push` - Apply the next patch in the series file.
- `quilt push -a` - Apply all patches in the series file.
- `quilt pop` - Remove the topmost patch from the stack of applied patches.
- `quilt pop -a` - Remove all applied patches.

With these commands, for example, you can apply all patches with `quilt push -a` or unapply them all with `quilt pop -a`. You can also verify what patches are applied with `quilt applied` or unapplied with `quilt unapplied`. `quilt push` applies the next unapplied patch (i.e. the patch returned by `quilt next`) and `quilt pop` unapplies the last applied patch (i.e. the patch returned by `quilt top`). You can give a patch name as parameter to `quilt push` or `quilt pop` and it will apply/unapply all the patches required until the given patch is on the top.

Example:

```sh
quilt pop -a
# Removing patch patches/example-patch-2.diff
# Removing example-file-2
# Restoring example-file-1
#
# Removing patch patches/example-patch-1.diff
# Restoring example-file-1
#
# No patches applied


quilt applied
# No patches applied


quilt next
# patches/example-patch-1.diff


quilt push -a
# Applying patch patches/example-patch-1.diff
# patching file example-file-1
#
# Applying patch patches/example-patch-2.diff
# patching file example-file-1
# patching file example-file-2
#
# Now at patch patches/example-patch-2.diff


quilt pop
# Removing patch patches/example-patch-2.diff
# Removing example-file-2
# Restoring example-file-1
#
# Now at patch patches/example-patch-1.diff


quilt push
# Applying patch patches/example-patch-2.diff
# patching file example-file-1
# patching file example-file-2
#
# Now at patch patches/example-patch-2.diff
```

### (7) Getting information about patches and files

We have the following commands:

- `quilt header [patch]` - Print or change the header of the topmost or specified patch.
- `quilt diff [-P patch] [file ...]` - Produces a diff of the specified file(s) in the topmost or specified patch. If no files are specified, all files that are modified are included.
- `quilt files [patch]` - Print the list of files that the topmost or specified patch changes.
- `quilt patches {file}` - Print the list of patches that modify the specified file (uses a heuristic to determine which files are modified by unapplied patches. Note that this heuristic is much slower than scanning applied patches).
- `quilt annotate [-P patch] {file}` - Print an annotated listing of the specified file showing which patches modify which lines. Only applied patches are included.

Example:

```sh
quilt annotate example-file-1
#         line 1
# 1       edit 2
#         line 3
# 1       new line 4
# 2       Line created by example-patch-2.diff
#
# 1       patches/example-patch-1.diff
# 2       patches/example-patch-2.diff
```

## Insert a template with DEP-3 headers

For a Debian package manintainer, `quilt header --dep3 -e [patch]` could be used to add DEP-3 meta-information to the header of the topmost or specified patch.

See [DEP-3 - Patch Tagging Guidelines](http://dep.debian.net/deps/dep3/) for details.

## Managing patches with 'gbp pq'

The `gbp pq` tool (included in 'git-buildpackage') can be used to manage a quilt series of patches against the upstream source while using Git to edit the packages when needed.

TODO.

Read [Debian New Maintainers' Guide - Chapter 3. Modifying the source](https://www.debian.org/doc/manuals/maint-guide/modify.en.html) and [Russ Allbery's - Using Git for Debian Packaging](https://www.eyrie.org/~eagle/notes/debian/git.html) for examples.
