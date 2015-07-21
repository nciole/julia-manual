
# 开发扩展包


Julia's package manager is designed so that when you have a package installed, you are already in a position to look at its source code and full development history.
You are also able to make changes to packages, commit them using git, and easily contribute fixes and enhancements upstream.
Similarly, the system is designed so that if you want to create a new package, the simplest way to do so is within the infrastructure provided by the package manager.

## Initial Setup


Since packages are git repositories, before doing any package development you should setup the following standard global git configuration settings::

```
    $ git config --global user.name "FULL NAME"
    $ git config --global user.email "EMAIL"
```

where ``FULL NAME`` is your actual full name (spaces are allowed between the double quotes) and ``EMAIL`` is your actual email address.
Although it isn't necessary to use `GitHub <https://github.com/>`_ to create or publish Julia packages, most Julia packages as of writing this are hosted on GitHub and the package manager knows how to format origin URLs correctly and otherwise work with the service smoothly.
We recommend that you create a `free account <https://github.com/join>`_ on GitHub and then do:

```
    $ git config --global github.user "USERNAME"
```

where ``USERNAME`` is your actual GitHub user name.
Once you do this, the package manager knows your GitHub user name and can configure things accordingly.
You should also `upload <https://github.com/settings/ssh>`_ your public SSH key to GitHub and set up an `SSH agent <http://linux.die.net/man/1/ssh-agent>`_ on your development machine so that you can push changes with minimal hassle.
In the future, we will make this system extensible and support other common git hosting options like `BitBucket <https://bitbucket.org>`_ and allow developers to choose their favorite.

## Generating a New Package


Suppose you want to create a new Julia package called ``FooBar``.
To get started, do ``Pkg.generate(pkg,license)`` where ``pkg`` is the new package name and ``license`` is the name of a license that the package generator knows about:

```
    julia> Pkg.generate("FooBar","MIT")
    INFO: Initializing FooBar repo: /Users/stefan/.julia/v0.3/FooBar
    INFO: Origin: git://github.com/StefanKarpinski/FooBar.jl.git
    INFO: Generating LICENSE.md
    INFO: Generating README.md
    INFO: Generating src/FooBar.jl
    INFO: Generating test/runtests.jl
    INFO: Generating .travis.yml
    INFO: Committing FooBar generated files
```

This creates the directory ``~/.julia/v0.3/FooBar``, initializes it as a git repository, generates a bunch of files that all packages should have, and commits them to the repository:

```
    $ cd ~/.julia/v0.3/FooBar && git show --stat

    commit 84b8e266dae6de30ab9703150b3bf771ec7b6285
    Author: Stefan Karpinski <stefan@karpinski.org>
    Date:   Wed Oct 16 17:57:58 2013 -0400

        FooBar.jl generated files.

            license: MIT
            authors: Stefan Karpinski
            years:   2013
            user: StefanKarpinski

        Julia Version 0.3.0-prerelease+3217 [5fcfb13*]

     .travis.yml      | 16 +++++++++++++
     LICENSE.md       | 22 +++++++++++++++++++++++
     README.md        |  3 +++
     src/FooBar.jl    |  5 +++++
     test/runtests.jl |  5 +++++
     5 files changed, 51 insertions(+)
```

At the moment, the package manager knows about the MIT "Expat" License, indicated by ``"MIT"``, the Simplified BSD License, indicated by ``"BSD"``, and version 2.0 of the Apache Software License, indicated by ``"ASL"``.
If you want to use a different license, you can ask us to add it to the package generator, or just pick one of these three and then modify the ``~/.julia/v0.3/PACKAGE/LICENSE.md`` file after it has been generated.

If you created a GitHub account and configured git to know about it,
``Pkg.generate`` will set an appropriate origin URL for you.  It will
also automatically generate a ``.travis.yml`` file for using the
`Travis <https://travis-ci.org>`_ automated testing service.  You will
have to enable testing on the Travis website for your package
repository, but once you've done that, it will already have working
tests.  Of course, all the default testing does is verify that ``using
FooBar`` in Julia works.

## Making Your Package Available

Once you've made some commits and you're happy with how ``FooBar`` is working, you may want to get some other people to try it out.
First you'll need to create the remote repository and push your code to it;
we don't yet automatically do this for you, but we will in the future and it's not too hard to figure out [3]_.
Once you've done this, letting people try out your code is as simple as sending them the URL of the published repo – in this case::

```
    git://github.com/StefanKarpinski/FooBar.jl.git
```

For your package, it will be your GitHub user name and the name of your package, but you get the idea.
People you send this URL to can use ``Pkg.clone`` to install the package and try it out:

```
    julia> Pkg.clone("git://github.com/StefanKarpinski/FooBar.jl.git")
    INFO: Cloning FooBar from git@github.com:StefanKarpinski/FooBar.jl.git
```

[3]: Installing and using GitHub's `"hub" tool
       <https://github.com/github/hub>`_ is highly recommended. It
       allows you to do things like run ``hub create`` in the package
       repo and have it automatically created via GitHub's API.

## Publishing Your Package


Once you've decided that ``FooBar`` is ready to be registered as an official package, you can add it to your local copy of ``METADATA`` using ``Pkg.register``:

```
    julia> Pkg.register("FooBar")
    INFO: Registering FooBar at git://github.com/StefanKarpinski/FooBar.jl.git
    INFO: Committing METADATA for FooBar
```

This creates a commit in the ``~/.julia/v0.3/METADATA`` repo:

```
    $ cd ~/.julia/v0.3/METADATA && git show

    commit 9f71f4becb05cadacb983c54a72eed744e5c019d
    Author: Stefan Karpinski <stefan@karpinski.org>
    Date:   Wed Oct 16 18:46:02 2013 -0400

        Register FooBar

    diff --git a/FooBar/url b/FooBar/url
    new file mode 100644
    index 0000000..30e525e
    --- /dev/null
    +++ b/FooBar/url
    @@ -0,0 +1 @@
    +git://github.com/StefanKarpinski/FooBar.jl.git
```

This commit is only locally visible, however.  In order to make it visible to
the world, you need to merge your local ``METADATA`` upstream into the official
repo.  The ``Pkg.publish()`` command will fork the ``METADATA`` repository on
GitHub, push your changes to your fork, and open a pull request ::

```
  julia> Pkg.publish()
  INFO: Validating METADATA
  INFO: No new package versions to publish
  INFO: Submitting METADATA changes
  INFO: Forking JuliaLang/METADATA.jl to StefanKarpinski
  INFO: Pushing changes as branch pull-request/ef45f54b
  INFO: To create a pull-request open:

    https://github.com/StefanKarpinski/METADATA.jl/compare/pull-request/ef45f54b
```

For various reasons ``Pkg.publish()`` sometimes does not succeed.  In
those cases, you may make a pull request on GitHub, which is [not
difficult](https://help.github.com/articles/creating-a-pull-request).

Once the package URL for ``FooBar`` is registered in the official ``METADATA`` repo, people know where to clone the package from, but there still aren't any registered versions available.
This means that ``Pkg.add("FooBar")`` won't work yet since it only installs official versions.
``Pkg.clone("FooBar")`` without having to specify a URL for it.
Moreover, when they run ``Pkg.update()``, they will get the latest version of ``FooBar`` that you've pushed to the repo.
This is a good way to have people test out your packages as you work on them, before they're ready for an official release.

## Tagging Package Versions


Once you are ready to make an official version your package, you can tag and register it with the ``Pkg.tag`` command:

```
    julia> Pkg.tag("FooBar")
    INFO: Tagging FooBar v0.0.1
    INFO: Committing METADATA for FooBar
```

This tags ``v0.0.1`` in the ``FooBar`` repo:

```
    $ cd ~/.julia/v0.3/FooBar && git tag
    v0.0.1
```

It also creates a new version entry in your local ``METADATA`` repo for ``FooBar``:

```
    $ cd ~/.julia/v0.3/FooBar && git show
    commit de77ee4dc0689b12c5e8b574aef7f70e8b311b0e
    Author: Stefan Karpinski <stefan@karpinski.org>
    Date:   Wed Oct 16 23:06:18 2013 -0400

        Tag FooBar v0.0.1

    diff --git a/FooBar/versions/0.0.1/sha1 b/FooBar/versions/0.0.1/sha1
    new file mode 100644
    index 0000000..c1cb1c1
    --- /dev/null
    +++ b/FooBar/versions/0.0.1/sha1
    @@ -0,0 +1 @@
    +84b8e266dae6de30ab9703150b3bf771ec7b6285
```

If there is a ``REQUIRE`` file in your package repo, it will be copied into the appropriate spot in ``METADATA`` when you tag a version.
Package developers should make sure that the ``REQUIRE`` file in their package correctly reflects the requirements of their package, which will automatically flow into the official metadata if you're using ``Pkg.tag``.
See the `Requirements Specification <#man-package-requirements>`_ for the full format of ``REQUIRE``.

The ``Pkg.tag`` command takes an optional second argument that is either an explicit version number object like ``v"0.0.1"`` or one of the symbols ``:patch``, ``:minor`` or ``:major``.
These increment the patch, minor or major version number of your package intelligently.

As with ``Pkg.register``, these changes to ``METADATA`` aren't
available to anyone else until they've been included upstream.  Again,
use the ``Pkg.publish()`` command, which first makes sure that
individual package repos have been tagged, pushes them if they haven't
already been, and then opens a pull request to ``METADATA``::

```
  julia> Pkg.publish()
  INFO: Validating METADATA
  INFO: Pushing FooBar permanent tags: v0.0.1
  INFO: Submitting METADATA changes
  INFO: Forking JuliaLang/METADATA.jl to StefanKarpinski
  INFO: Pushing changes as branch pull-request/3ef4f5c4
  INFO: To create a pull-request open:
```
    https://github.com/StefanKarpinski/METADATA.jl/compare/pull-request/3ef4f5c4

## Fixing Package Requirements


If you need to fix the registered requirements of an already-published package version, you can do so just by editing the metadata for that version, which will still have the same commit hash – the hash associated with a version is permanent::

```
  $ cd ~/.julia/v0.3/METADATA/FooBar/versions/0.0.1 && cat requires
  julia 0.3-

  $ vi requires
```

Since the commit hash stays the same, the contents of the ``REQUIRE`` file that will be checked out in the repo will **not** match the requirements in ``METADATA`` after such a change;
this is unavoidable.
When you fix the requirements in ``METADATA`` for a previous version of a package, however, you should also fix the ``REQUIRE`` file in the current version of the package.



## 依赖关系


The ``~/.julia/v0.3/REQUIRE`` file, the ``REQUIRE`` file inside packages, and the ``METADATA`` package ``requires`` files use a simple line-based format to express the ranges of package versions which need to be installed.  Package ``REQUIRE`` and ``METADATA requires`` files should also include the range of versions of ``julia`` the package is expected to work with.

Here's how these files are parsed and interpreted.

* Everything after a ``#`` mark is stripped from each line as a comment.
* If nothing but whitespace is left, the line is ignored.
* If there are non-whitespace characters remaining, the line is a requirement and the is split on whitespace into words.

The simplest possible requirement is just the name of a package name on a line by itself:

```
    Distributions
```

This requirement is satisfied by any version of the ``Distributions`` package.
The package name can be followed by zero or more version numbers in ascending order, indicating acceptable intervals of versions of that package.
One version opens an interval, while the next closes it, and the next opens a new interval, and so on;
if an odd number of version numbers are given, then arbitrarily large versions will satisfy;
if an even number of version numbers are given, the last one is an upper limit on acceptable version numbers.
For example, the line:

```
    Distributions 0.1
```

is satisfied by any version of ``Distributions`` greater than or equal to ``0.1.0``.
Suffixing a version with `-` allows any pre-release versions as well. For example:

```
    Distributions 0.1-
```

is satisfied by pre-release versions such as ``0.1-dev`` or ``0.1-rc1``, or by any version greater than or equal to ``0.1.0``.

This requirement entry:

```
    Distributions 0.1 0.2.5
```

is satisfied by versions from ``0.1.0`` up to, but not including ``0.2.5``.
If you want to indicate that any ``0.1.x`` version will do, you will want to write:

```
    Distributions 0.1 0.2-
```

If you want to start accepting versions after ``0.2.7``, you can write::

```
    Distributions 0.1 0.2- 0.2.7
```

If a requirement line has leading words that begin with ``@``, it is a system-dependent requirement.
If your system matches these system conditionals, the requirement is included, if not, the requirement is ignored.
For example:

```
    @osx Homebrew
```

will require the ``Homebrew`` package only on systems where the operating system is OS X.
The system conditions that are currently supported are:

```
    @windows
    @unix
    @osx
    @linux
```

The ``@unix`` condition is satisfied on all UNIX systems, including OS X, Linux and FreeBSD.
Negated system conditionals are also supported by adding a ``!`` after the leading ``@``.
Examples:

```
    @!windows
    @unix @!osx
```

The first condition applies to any system but Windows and the second condition applies to any UNIX system besides OS X.

Runtime checks for the current version of Julia can be made using the built-in
``VERSION`` variable, which is of type ``VersionNumber``. Such code is
occasionally necessary to keep track of new or deprecated functionality between
various releases of Julia. Examples of runtime checks:

```
    VERSION < v"0.3-" #exclude all pre-release versions of 0.3

    v"0.2-" <= VERSION < v"0.3-" #get all 0.2 versions, including pre-releases, up to the above

    v"0.2" <= VERSION < v"0.3-" #To get only stable 0.2 versions (Note v"0.2" == v"0.2.0")

    VERSION >= v"0.2.1" #get at least version 0.2.1
```

See the section on :ref:`version number literals <man-version-number-literals>` for a more complete description.

