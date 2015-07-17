
# 扩展包


Julia 内置了一个包管理系统，可以用这个系统来完成包的管理，当然，你也可以用你的操作系统自带的，或者从源码编译。
你可以在 http://pkg.julialang.org  找到所有已注册（一种发布包的机制）的包的列表。
所有的包管理命令都包含在 ``Pkg`` 这个module里面，Julia的 Base install 引入了 ``Pkg`` 。

扩展包状态
----------

可以通过 ``Pkg.status()`` 这个方程，打印出一个你所有安装的包的总结。

刚开始的时候，你没有安装任何包::

    julia> Pkg.status()
    INFO: Initializing package repository /Users/stefan/.julia/v0.3
    INFO: Cloning METADATA from git://github.com/JuliaLang/METADATA.jl
    No packages installed.

当你第一次运行 ``Pkg`` 的一个命令时， 你的包目录（所有的包被安装在一个统一的目录下）会自动被初始化，因为 ``Pkg`` 希望有这样一个目录，这个目录的信息被包含于 ``Pkg.status()`` 中。

这里是一个简单的，已经有少量被安装的包的例子::

    julia> Pkg.status()
    Required packages:
     - Distributions                 0.2.8
     - UTF16                         0.2.0
    Additional packages:
     - NumericExtensions             0.2.17
     - Stats                         0.2.6

这些包，都是已注册了的版本，并且通过 ``Pkg`` 管理。
安装了的包可以是一个更复杂的"状态"，通过"注释"来表明正确的版本；当我们遇到这些“状态”和“注释”时我们会解释的。

为了编程需要，``Pkg.installed()`` 返回一个字典，这个字典对应了安装了的包的名字和其现在使用的版本::

    julia> Pkg.installed()
    ["Distributions"=>v"0.2.8","Stats"=>v"0.2.6","UTF16"=>v"0.2.0","NumericExtensions"=>v"0.2.17"]

添加和删除扩展包
----------------

Julia's package manager is a little unusual in that it is declarative rather than imperative.
This means that you tell it what you want and it figures out what versions to install (or remove) to satisfy those requirements optimally – and minimally.
So rather than installing a package, you just add it to the list of requirements and then "resolve" what needs to be installed.
In particular, this means that if some package had been installed because it was needed by a previous version of something you wanted, and a newer version doesn't have that requirement anymore, updating will actually remove that package.

Your package requirements are in the file ``~/.julia/v0.3/REQUIRE``.
You can edit this file by hand and then call ``Pkg.resolve()`` to install, upgrade or remove packages to optimally satisfy the requirements, or you can do ``Pkg.edit()``, which will open ``REQUIRE`` in your editor (configured via the ``EDITOR`` or ``VISUAL`` environment variables), and then automatically call ``Pkg.resolve()`` afterwards if necessary.
If you only want to add or remove the requirement for a single package, you can also use the non-interactive ``Pkg.add`` and ``Pkg.rm`` commands, which add or remove a single requirement to ``REQUIRE`` and then call ``Pkg.resolve()``.

You can add a package to the list of requirements with the ``Pkg.add`` function, and the package and all the packages that it depends on will be installed:

    julia> Pkg.status()
    No packages installed.

    julia> Pkg.add("Distributions")
    INFO: Cloning cache of Distributions from git://github.com/JuliaStats/Distributions.jl.git
    INFO: Cloning cache of NumericExtensions from git://github.com/lindahua/NumericExtensions.jl.git
    INFO: Cloning cache of Stats from git://github.com/JuliaStats/Stats.jl.git
    INFO: Installing Distributions v0.2.7
    INFO: Installing NumericExtensions v0.2.17
    INFO: Installing Stats v0.2.6
    INFO: REQUIRE updated.

    julia> Pkg.status()
    Required packages:
     - Distributions                 0.2.7
    Additional packages:
     - NumericExtensions             0.2.17
     - Stats                         0.2.6

What this is doing is first adding ``Distributions`` to your ``~/.julia/v0.3/REQUIRE`` file:

    $ cat ~/.julia/v0.3/REQUIRE
    Distributions

It then runs ``Pkg.resolve()`` using these new requirements, which leads to the conclusion that the ``Distributions`` package should be installed since it is required but not installed.
As stated before, you can accomplish the same thing by editing your ``~/.julia/v0.3/REQUIRE`` file by hand and then running ``Pkg.resolve()`` yourself:

    $ echo UTF16 >> ~/.julia/v0.3/REQUIRE

    julia> Pkg.resolve()
    INFO: Cloning cache of UTF16 from git://github.com/nolta/UTF16.jl.git
    INFO: Installing UTF16 v0.2.0

    julia> Pkg.status()
    Required packages:
     - Distributions                 0.2.7
     - UTF16                         0.2.0
    Additional packages:
     - NumericExtensions             0.2.17
     - Stats                         0.2.6

This is functionally equivalent to calling ``Pkg.add("UTF16")``,
except that ``Pkg.add`` doesn't change ``REQUIRE`` until *after*
installation has completed, so if there are problems, ``REQUIRE`` will
be left as it was before calling ``Pkg.add``.  The format of the
``REQUIRE`` file is described in `Requirements Specification`_; it
allows, among other things, requiring specific ranges of versions of
packages.

When you decide that you don't want to have a package around any more,
you can use ``Pkg.rm`` to remove the requirement for it from the
``REQUIRE`` file:

    julia> Pkg.rm("Distributions")
    INFO: Removing Distributions v0.2.7
    INFO: Removing Stats v0.2.6
    INFO: Removing NumericExtensions v0.2.17
    INFO: REQUIRE updated.

    julia> Pkg.status()
    Required packages:
     - UTF16                         0.2.0

    julia> Pkg.rm("UTF16")
    INFO: Removing UTF16 v0.2.0
    INFO: REQUIRE updated.

    julia> Pkg.status()
    No packages installed.

Once again, this is equivalent to editing the ``REQUIRE`` file to remove the line with each package name on it then running ``Pkg.resolve()`` to update the set of installed packages to match.
While ``Pkg.add`` and ``Pkg.rm`` are convenient for adding and removing requirements for a single package, when you want to add or remove multiple packages, you can call ``Pkg.edit()`` to manually change the contents of ``REQUIRE`` and then update your packages accordingly.
``Pkg.edit()`` does not roll back the contents of ``REQUIRE`` if ``Pkg.resolve()`` fails – rather, you have to run ``Pkg.edit()`` again to fix the files contents yourself.

Because the package manager uses git internally to manage the package git repositories, users may run into protocol issues (if behind a firewall, for example), when running ``Pkg.add``. The following command can be run from the command line to tell git to use 'https' instead of the 'git' protocol when cloning repositories:

    git config --global url."https://".insteadOf git://

安装未注册的扩展包
------------------

Julia packages are simply git repositories, clonable via any of the [protocols](https://www.kernel.org/pub/software/scm/git/docs/git-clone.html#URLS) that git supports, and containing Julia code that follows certain layout conventions.
Official Julia packages are registered in the [METADATA.jl](https://github.com/JuliaLang/METADATA.jl) repository, available at a well-known location [1]_.
The ``Pkg.add`` and ``Pkg.rm`` commands in the previous section interact with registered packages, but the package manager can install and work with unregistered packages too.
To install an unregistered package, use ``Pkg.clone(url)``, where ``url`` is a git URL from which the package can be cloned:

    julia> Pkg.clone("git://example.com/path/to/Package.jl.git")
    INFO: Cloning Package from git://example.com/path/to/Package.jl.git
    Cloning into 'Package'...
    remote: Counting objects: 22, done.
    remote: Compressing objects: 100% (10/10), done.
    remote: Total 22 (delta 8), reused 22 (delta 8)
    Receiving objects: 100% (22/22), 2.64 KiB, done.
    Resolving deltas: 100% (8/8), done.

By convention, Julia repository names end with ``.jl`` (the additional ``.git`` indicates a "bare" git repository), which keeps them from colliding with repositories for other languages, and also makes Julia packages easy to find in search engines.
When packages are installed in your ``.julia/v0.3`` directory, however, the extension is redundant so we leave it off.

If unregistered packages contain a ``REQUIRE`` file at the top of their source tree, that file will be used to determine which registered packages the unregistered package depends on, and they will automatically be installed.
Unregistered packages participate in the same version resolution logic as registered packages, so installed package versions will be adjusted as necessary to satisfy the requirements of both registered and unregistered packages.

[1] The official set of packages is at https://github.com/JuliaLang/METADATA.jl, but individuals and organizations can easily use a different metadata repository. This allows control which packages are available for automatic installation. One can allow only audited and approved package versions, and make private packages or forks available.

更新扩展包
----------

When package developers publish new registered versions of packages that you're using, you will, of course, want the new shiny versions.
To get the latest and greatest versions of all your packages, just do ``Pkg.update()``:

    julia> Pkg.update()
    INFO: Updating METADATA...
    INFO: Computing changes...
    INFO: Upgrading Distributions: v0.2.8 => v0.2.10
    INFO: Upgrading Stats: v0.2.7 => v0.2.8

The first step of updating packages is to pull new changes to ``~/.julia/v0.3/METADATA`` and see if any new registered package versions have been published.
After this, ``Pkg.update()`` attempts to update packages that are checked out on a branch and not dirty (i.e. no changes have been made to files tracked by git) by pulling changes from the package's upstream repository.
Upstream changes will only be applied if no merging or rebasing is necessary – i.e. if the branch can be `"fast-forwarded" <http://git-scm.com/book/en/Git-Branching-Basic-Branching-and-Merging>`_.
If the branch cannot be fast-forwarded, it is assumed that you're working on it and will update the repository yourself.

Finally, the update process recomputes an optimal set of package versions to have installed to satisfy your top-level requirements and the requirements of "fixed" packages.
A package is considered fixed if it is one of the following:

1. **Unregistered:** the package is not in ``METADATA`` – you installed it with ``Pkg.clone``.
2. **Checked out:** the package repo is on a development branch.
3. **Dirty:** changes have been made to files in the repo.

If any of these are the case, the package manager cannot freely change the installed version of the package, so its requirements must be satisfied by whatever other package versions it picks.
The combination of top-level requirements in ``~/.julia/v0.3/REQUIRE`` and the requirement of fixed packages are used to determine what should be installed.

Checkout, Pin and Free
----------------------

You may want to use the ``master`` version of a package rather than one of its registered versions.
There might be fixes or functionality on master that you need that aren't yet published in any registered versions, or you may be a developer of the package and need to make changes on ``master`` or some other development branch.
In such cases, you can do ``Pkg.checkout(pkg)`` to checkout the ``master`` branch of ``pkg`` or ``Pkg.checkout(pkg,branch)`` to checkout some other branch::

    julia> Pkg.add("Distributions")
    INFO: Installing Distributions v0.2.9
    INFO: Installing NumericExtensions v0.2.17
    INFO: Installing Stats v0.2.7
    INFO: REQUIRE updated.

    julia> Pkg.status()
    Required packages:
     - Distributions                 0.2.9
    Additional packages:
     - NumericExtensions             0.2.17
     - Stats                         0.2.7

    julia> Pkg.checkout("Distributions")
    INFO: Checking out Distributions master...
    INFO: No packages to install, update or remove.

    julia> Pkg.status()
    Required packages:
     - Distributions                 0.2.9+             master
    Additional packages:
     - NumericExtensions             0.2.17
     - Stats                         0.2.7

Immediately after installing ``Distributions`` with ``Pkg.add`` it is on the current most recent registered version – ``0.2.9`` at the time of writing this.
Then after running ``Pkg.checkout("Distributions")``, you can see from the output of ``Pkg.status()`` that ``Distributions`` is on an unregistered version greater than ``0.2.9``, indicated by the "pseudo-version" number ``0.2.9+``.

When you checkout an unregistered version of a package, the copy of the ``REQUIRE`` file in the package repo takes precedence over any requirements registered in ``METADATA``, so it is important that developers keep this file accurate and up-to-date, reflecting the actual requirements of the current version of the package.
If the ``REQUIRE`` file in the package repo is incorrect or missing, dependencies may be removed when the package is checked out.
This file is also used to populate newly published versions of the package if you use the API that ``Pkg`` provides for this (described below).

When you decide that you no longer want to have a package checked out on a branch, you can "free" it back to the control of the package manager with ``Pkg.free(pkg)``:

    julia> Pkg.free("Distributions")
    INFO: Freeing Distributions...
    INFO: No packages to install, update or remove.

    julia> Pkg.status()
    Required packages:
     - Distributions                 0.2.9
    Additional packages:
     - NumericExtensions             0.2.17
     - Stats                         0.2.7

After this, since the package is on a registered version and not on a branch, its version will be updated as new registered versions of the package are published.

If you want to pin a package at a specific version so that calling ``Pkg.update()`` won't change the version the package is on, you can use the ``Pkg.pin`` function::

    julia> Pkg.pin("Stats")
    INFO: Creating Stats branch pinned.47c198b1.tmp

    julia> Pkg.status()
    Required packages:
     - Distributions                 0.2.9
    Additional packages:
     - NumericExtensions             0.2.17
     - Stats                         0.2.7              pinned.47c198b1.tmp

After this, the ``Stats`` package will remain pinned at version ``0.2.7`` – or more specifically, at commit ``47c198b1``, but since versions are permanently associated a given git hash, this is the same thing.
``Pkg.pin`` works by creating a throw-away branch for the commit you want to pin the package at and then checking that branch out.
By default, it pins a package at the current commit, but you can choose a different version by passing a second argument:

    julia> Pkg.pin("Stats",v"0.2.5")
    INFO: Creating Stats branch pinned.1fd0983b.tmp
    INFO: No packages to install, update or remove.

    julia> Pkg.status()
    Required packages:
     - Distributions                 0.2.9
    Additional packages:
     - NumericExtensions             0.2.17
     - Stats                         0.2.5              pinned.1fd0983b.tmp

Now the ``Stats`` package is pinned at commit ``1fd0983b``, which corresponds to version ``0.2.5``.
When you decide to "unpin" a package and let the package manager update it again, you can use ``Pkg.free`` like you would to move off of any branch::

    julia> Pkg.free("Stats")
    INFO: Freeing Stats...
    INFO: No packages to install, update or remove.

    julia> Pkg.status()
    Required packages:
     - Distributions                 0.2.9
    Additional packages:
     - NumericExtensions             0.2.17
     - Stats                         0.2.7

After this, the ``Stats`` package is managed by the package manager again, and future calls to ``Pkg.update()`` will upgrade it to newer versions when they are published.
The throw-away ``pinned.1fd0983b.tmp`` branch remains in your local ``Stats`` repo, but since git branches are extremely lightweight, this doesn't really matter;
if you feel like cleaning them up, you can go into the repo and delete those branches.

 [2] :Packages that aren't on branches will also be marked as dirty if you make changes in the repo, but that's a less common thing to do.

