# Python

Whether you distribute your application from the [Python Package Index](https://pypi.python.org) or alternative means, Snapcraft builds on top of your existing release process to offer easy installation and automatic, non-interactive updates.

Its isolation model and depdendency bundling save you from relying on the distribution library packages which cannot be pinned by your application to specific known-to-work versions. Nor do you need to resort to asking users to install and manage virtualenv.

## Getting started

The python plugin works with either python 2 or 3 based applications. It can:

- Build a python project that has a setup.py
- Import python modules specified in requirements.txt
- Or install packages straight from pip

Let's look at OfflineIMAP, a python 2 application that downloads your mailboxes.
```yaml
name: offlineimap
version: '7.0.14'
summary: Offline IMAP
description: |
  Read/sync your IMAP mailboxes
confinement: strict

apps:
  offlineimap:
    command: bin/offlineimap
    plugs: [network]

parts:
  offlineimap:
    plugin: python
    python-version: python2
    source: .
    source-tag: 'v7.0.14'
    build-packages: [build-essential]
```

We explicitly specify the python version (`python-version`). OfflineIMAP ships a requirements.txt file, which is automatically detected and used by the plugin. However, if your project uses a custom name for the requirements file (e.g. requirements26.txt), you can use the `requirements:` option.

You can build this example yourself by running:
```sh
$ git clone https://github.com/popey/offlineimap
$ cd offlineimap
$ snapcraft
$ sudo snap install offlineimap_*.snap --dangerous
```

## Further customisation
The python plugin provides these additional options.

```
- requirements:
  (string)
  Path to a requirements.txt file
- constraints:
  (string)
  Path to a constraints file
- process-dependency-links:
  (bool; default: false)
  Enable the processing of dependency links.
- python-packages:
  (list)
  A list of dependencies to get from PyPi
- python-version:
  (string; default: python3)
  The python version to use. Valid options are: python2 and python3
  ```

  Additional shell commands can be run between steps using the [prepare, build, or install keywords](https://snapcraft.io/docs/reference/plugins/common).
