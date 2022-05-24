# Commands for analysis and troubleshooting java projects

## Print content of an JAR archive

This command lists all files inside a JAR archive:

```sh
jar -tvf <JARFILE>
```

The meaning of each parameter is:
- `-t or --list` - Lists the table of contents for the archive.
- `-v or --verbose` - Sends or prints verbose output to standard output.
- `-f=FILE or --file=FILE` - Specifies the archive file name.

## Search for a dependency with a class name

You can search on [Maven Central](https://search.maven.org/) for a package containing a class name putting for example `c:JUnit4` for a simple search or for example `fc:org.sonatype.nexus` for a more precise result using a full class name.

## Search a class file inside a directory with JAR archives

NOTE: this command could take a long time (minutes) trying to find the file.

The following command searches in `<DIRECTORY>` all the JAR archives containing a file with the `<CLASSNAME>` pattern:

```sh
find <DIRECTORY> -name "*.jar" -exec sh -c "jar -tf {} | egrep -is '<CLASSNAME>' | sed 's|^|{}:|'" \;
```

The meaning of each parameter for grep is:
- `-i, --ignore-case` - Ignore case distinctions in both the PATTERN and the input files.
- `-s, --no-messages` - Suppress error messages about nonexistent or unreadable files.

Taking into account that in Java every package name of a class corresponds to a file path, we could for example search the JAR archive where the 'org.springframework.http.HttpStatus' class is stored with the following command:

```sh
find ~/.m2/repository/org/springframework/ -name "*.jar" -exec sh -c "jar -tf {} | grep -is 'org/springframework/http/HttpStatus.class' | sed 's|^|{}:|'" \;
```

I replaced egrep with grep because I want to search an exact value.

# How to debug

You can attach a debugger to a running process. The following steps configures IntelliJ IDEA as the debugger.

1. Run > Edit Configurations
2. Click "+" and select "Remote JVM Debug"
3. Put a convenient name
4. Copy the text inside "Command line arguments for remote JVM" and add append it to the command that will execute the code.
```
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```
5. Click "OK" to save

If you want your application to wait until debugger is connected just change suspend flag to 'suspend=y'.

NOTE: if you stop the debugger session the code will still continue his execution.
