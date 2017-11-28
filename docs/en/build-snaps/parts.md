---
layout: base
title: Parts
---

Parts are reusable components that are the main building blocks used to create snaps using Snapcraft. Parts have their own private space and lifecycle. Each part uses a plugin, which tells the part how to behave and what to do with the information inside it. Parts are analogous to a library that you would call in your program. There are three types of parts:

- Parts that use local files on your machine. For example:

         parts:
           hello:
             plugin: nodejs
               source: .

- Parts from online sources, such as `git`, `bzr`, a tarball, or any code repository you like. For example:

        parts:
          godd:
           plugin: go
            source: https://github.com/mvo5/godd.git
          gnu-hello:
           plugin: autotools
            source: http://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz

- Parts built and shared by others through the [parts repository](https://wiki.ubuntu.com/snapcraft/parts). For example, using a part for `curl` defined in this repository:

        parts:
            client:
               plugin: autotools
               source: .
               after: [curl]

**Tips**

* Run `snapcraft update` to keep your shared parts list synchronized with the parts repository
* Run `snapcraft search` to list all parts from the repository
* Run `snapcraft define PART` to show a shared part, including help for the part

## Defining parts in snapcraft.yaml

The snapcraft.yaml `parts` key defines a map of the parts you want to include in your snap. Each plugin used for a part provides his own set of rules and sub-keys, but also relies on a common set of sub-keys:

Key | Type | Purpose
:----- | :---- | :-----
`plugin` | (string) | Specifies the plugin that will manage this part.  Snapcraft will pass the plugin all the other user-specified part options, those options defined with the other keys below. There are three ways in which the plugin can be defined: a plugin name to use a built-in plug-in (for details, see the [list of built-in plugins](/reference/plugins)), a local path such as `parts/plugins/x-plugin_name.py` to use a local (custom defined) plugin and if `plugin` is not defined locally, the plugin defined for the part in the [parts repository](https://wiki.ubuntu.com/snapcraft/parts).
`after` | (list of strings) | Specifies any parts that should be built before this part, which Snapcraft then stages before trying to build this part.  This is useful when a part needs a library or build tool built by another part. If the part defined in `after` is not defined locally, the part will be searched for in the [parts repository](https://wiki.ubuntu.com/snapcraft/parts).
`stage-packages` | (list of strings) | A list of Ubuntu packages to use that are needed to support the part creation.
`filesets` | (yaml subsection) | A dictionary with filesets, the key being a recognizable user defined string and its value a list of strings of files to be included or excluded. Globbing is achieved with `*` for either inclusions or exclusion. Exclusions are denoted by a `-`. Globbing is computed from the private sections of the part.
`organize` | (yaml subsection) | A dictionary exposing replacements. The key is the internal name while the value the exposed (replacement) name (for example, `source_name: map_name`). **Note**: `filesets` refer to the exposed names of files, after the organization has been applied.
`stage` | (list of strings) | A list of files from a part’s installation to expose in stage. Rules applying to the list here are the same as those of filesets. Referencing of fileset keys is done with a `$` prefixing the fileset key, which will expand with the value of such key.
`prime` | (list of strings) | A list of files from a part’s installation to expose in the snap. Rules applying to the list here are the same as those of filesets. Referencing of fileset keys is done with a `$` prefixing the fileset key, which will expand with the value of such key.

You can define your parts in any order, `after` takes care of any required build order.

Usage examples for these sub-keys are available in the [common keywords](/reference/plugins/common) reference documentation.

## Working with Snapcraft parts

You've a number of ways in which to organize and include parts into you snaps. For example, you may want to enhance your part’s functionality using `stage-packages` to bring Ubuntu deb-based packages into your part, `filesets` to declare inclusion and exclusion sets, `organize` to make the artifact output for your part neater, `stage` and `prime` to make certain only the right set of files is seen at each stage (making use of filesets or not).

### Advanced example

An example integrating all these concepts for a part called `example-part` using a plugin called `nil` would look like:

     parts:
       example-part:
         type: nil
         stage-packages:
           - gnupg
           - wget
         organize:
           usr/bin: bin
         filesets:
           binaries:
            - bin/*
            - usr/bin/*
           headers:
            - “*.h”
            - -include
         stage:
           - $binaries
           - test/bin/test_app
           - $headers
         prime:
           - $binaries

In this example

* the [`nil`](/reference/plugins/nil) plugin makes use of `stage-packages`, these packages are fetched from the Ubuntu deb archive using the release (eg. 16.04) that is being used on the host. In this case, the part will be enhanced by the `gpg` and `wget` deb packages and its necessary dependencies to work isolated inside the part.

* when reaching the stage phase, the components in the part’s install directory will be exposed there, but since the `organize` key is being used, the contents in the install directory will be exposed to other parts in a cleaner form if desired or required; it is important to notice that when using `filesets` they will follow the organized files and not the initial layout.

* the concept of `filesets` enables the creation of sets named after the keywords defined within, in this case binaries and headers, these are not necessarily needed but allow for variable expansion in the common targets: stage and snap. An inclusion is defined by just listing the target file, it can be globbed with "\*" and a file can be explicitly excluded by prepending a "-".

* The `stage` key then:

    - replaces `$binaries` with the binaries defined in filesets.
    - Adds `test/bin/test_app` to the stage directory.
    - `$headers` will include all the header files, except those from `include` as it is prefixed with -, indicating these files should be excluded.

    These are the files that will be moved to the stage directory.

    The behavior for the `prime` step is identical to `stage` except that the file manipulation is applied to the `prime` directory, which is the final file/content layout for the snap. This is where everything should look clean and crisp for a good quality snap.

## Snapcraft for Python with PIP

Snapcraft includes support for Python 2.x and Python 3.x parts; here’s how a snapcraft.yaml parts section will look:

     parts:
       spongeshaker:
         plugin: python
         python-version: python3
         source: git://github.com/markokr/spongeshaker.git

A Python part will typically make sure that the required Python packages are installed on the build host and embed the following pieces in your snap:

- latest Python runtime from the latest Ubuntu packages of your current Ubuntu release
- latest PIP for this Python version as downloaded from PyPy
- latest versions of your PIP requirements

The proper **PYTHONPATH** environment variable will also be set in the wrapper scripts generated by Snapcraft or when running your app locally.

Python parts support standard Snapcraft options and the requirements option to point PIP at its requirements file.

For details, see the complete reference and real-world examples of the [`python`](/reference/plugins/python) plugin.

### Embedding a Python runtime

While the Core snap includes a Python runtime, this might not be the one you need, and it might be updated to a different version or removed in a Core snap update. This is why applications using Python can embed their copy of the Python runtime.

## Snapcraft for Java, Maven or Ant

Snapcraft includes support for building parts with Apache Maven or Ant; here’s how a snapcraft.yaml parts section will look:

    parts:
        webapp:
            plugin: maven
            source: git://github.com/lool/snappy-mvn-demo.git

A Maven part will typically:

- Make sure the tool is installed on the build host.
- Embed a Java runtime in your snap.
- Run mvn package and copy the resulting `*.jar` and `*.war` files in your snaps `jar/` and `war/` directories.

An Ant part works similarly, except it runs ant and sets the proper **CLASSPATH** environment variable in the wrapper scripts generated by Snapcraft or when running the app locally.

If you only need to embed a Java runtime, add a part with the `jdk` type. This will pull a relocatable OpenJDK via the default-jdk Ubuntu package and will set the proper **JAVA_HOME** and **PATH** environment variables in wrapper scripts generated by Snapcraft or when running the app locally.

For details, see the complete reference and real-world examples of the [`maven`](/reference/plugins/maven) and [`ant`](/reference/plugins/ant) plugins.
