# QGIS Enhancement Proposal: **Turning Plugin Management into Actual Package Management**

<table>
    <tr><th>date</th><td> 2020-05-21</td></tr>
    <tr><th>author</th><td><a href="http://www.pleiszenburg.de/?qep">Sebastian M. Ernst</a> (<a href="https://github.com/s-m-e">@s-m-e</a>)</td></tr>
    <tr><th>contact</th><td>ernst@pleiszenburg.de</td></tr>
    <tr><th>maintainer</th><td><a href="https://github.com/s-m-e">@s-m-e</a></td></tr>
    <tr><th>version</th><td>QGIS 3.16</td></tr> <!-- TODO -->
</table>

# Abstract <!-- MUST -->

QGIS Python plugins can not explicitly depend on regular Python packages. Although QGIS Python plugins can depend on other QGIS Python plugins, introduced in QGIS 3.8, this mechanism is far away from mature. Code quality, design and maintainability of the entire current plugin management system within QGIS, based on a detailed analysis of version 3.12, are questionable at best. This document proposes (a) to re-implement the existing plugin management system with all of its features, (b) to clean up the cross-plugin dependency design and (c) to add support for both the `conda` and the `pip` Python package managers for managing QGIS Python plugins - effectively adding support for dependencies between QGIS Python plugins and regular Python packages. These proposed changes are fully backward compatible and do not introduce adverse performance characteristics.

# Table of Contents

1) [Motivation](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#1-motivation)
    - [Background](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#background)
    - [How does QGIS fit into the bigger picture?](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#how-does-qgis-fit-into-the-bigger-picture)
    - [Packaging Terminology](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#packaging-terminology)
    - [Currently Missing Links](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#currently-missing-links)
    - [Currently Unsupported Use-Cases](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#currently-unsupported-use-cases)
2) [Proposed, Preferred Solution](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#2-proposed-preferred-solution-)
    - [QGIS Plugin Management Terminology](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#qgis-plugin-management-terminology)
    - [Analysis of Current Implementation](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#analysis-of-current-implementation)
        - [Tasks](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#tasks)
        - [Structural Overview](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#structural-overview)
        - [State of the Code](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#state-of-the-code)
        - [Summary of Findings](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#summary-of-findings)
    - [Fundamental Design Constraints for a Potential Solution](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#fundamental-design-constraints-for-a-potential-solution)
    - [Implementation Details](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#implementation-details)
        - [Step 1: Fully backward-compatible re-implementation of the plugin management system](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#step-1-fully-backward-compatible-re-implementation-of-the-plugin-management-system)
        - [Step 2: Adding support for Python package managers](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#step-2-adding-support-for-python-package-managers)
        - [Code Structure](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#code-structure)
    - [Affected APIs](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#affected-apis)
    - [Affected Files](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#affected-files-)
    - [Recommended changes to related projects](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#recommended-changes-to-related-projects)
    - [Example](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#example-)
3) [Alternative, Unfavorable Solutions](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#3-alternative-unfavorable-solutions)
4) [Performance Implications](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#4-performance-implications-)
5) [Backward Compatibility](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#5-backward-compatibility-)
6) [Copyright](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#6-copyright)
7) [Issue Tracking ID(s)](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#7-issue-tracking-ids-)
8) [Votes](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#8-votes-)

# 1. Motivation

## Background

QGIS offers the possibility to implement plugins, both in C++ and in Python. While it can be considered complicated to implement and even harder to distribute C++ plugins, it is relatively easy to implement and distribute Python plugins. QGIS' Python APIs offer access to almost all relevant QGIS features. Therefore, QGIS's extensibility through Python can be understood as one of its greatest strengths.

As of May 2020, [Python is the third-most-popular programming language](https://www.tiobe.com/tiobe-index/) in active use. It is only outperformed by C and Java. In this context, Python is currently seeing a massive increase in adoption across many different types of applications and disciplines - in a lot of cases driving users and developers away from established domain-specific proprietary solutions and into open source. It is safe to say that Python did something that only very few open source projects and virtually no programming language on its own managed to do: It has diverted significant amounts of funding into open source. The Python package ecosystem reflects that.

Distributing Python packages is currently a non-trivial problem, unfortunately. It can be considered the Achilles' heel of the Python ecosystem. Fundamental, mission-critical tools such as [setuptools](https://github.com/pypa/setuptools) and [distutils](https://docs.python.org/3/library/distutils.html) are infamous for their complicated use as well as their incomplete and inconsistent documentation. Younger projects like [flit](https://github.com/takluyver/flit) try to offer modern packaging tools and good documentation but are lacking much desired features. Somewhat more established projects like [cookiecutter](https://cookiecutter.readthedocs.io/) try to address known shortcomings by offering Python project template engines as well as many useful Python project templates. The latter solves some problems but, above a certain degree of complexity, still requires deep background knowledge for actually being useful.

For the purpose of this discussion, Python packages can be divided into two categories: Packages without binary components and Packages with binary components. The first category is relatively easy to package and distribute across all relevant platforms. The second category is both problematic but also much more interesting. One of Python's strengths is its ability to play nice with all sorts of other programming languages, e.g. C, C++ and Fortran. It has therefore become natural to integrate components written in other programming languages into Python packages. A clear majority of contemporary scientific and geospatial Python packages are following this approach and greatly benefit from it. It allows all sorts of interesting applications, such as accessing established numerical libraries, actual thread-based parallelism (something the [CPython interpreter](https://en.wikipedia.org/wiki/CPython) itself [is not capable of](https://wiki.python.org/moin/GlobalInterpreterLock) at the moment) and [access to GPUs for general-purpose computing](https://en.wikipedia.org/wiki/General-purpose_computing_on_graphics_processing_units). The downside is that both packaging *and installing* Python packages of this kind usually requires rather significant toolchains and sufficient background knowledge, among other issues. In the Linux ecosystem, the mentioned tool chains (as well as the mentioned background knowledge) are rather common, so it is safe to say that many (but not all) Linux users will not have massive problems installing common scientific and geospatial Python packages. On OS X, the overall picture is somewhat similar. However, on Windows, things are known to be highly problematic.

The most widely used distribution channel for Python packages happens to be the [Python Package Index](https://pypi.org/). As of May 2020, it holds roughly 1/4 million packages. Python packages can be installed from the index through [`pip`](https://pip.pypa.io/en/stable/), a fully featured [package manager](https://en.wikipedia.org/wiki/Package_manager). Both `pip` and the index can handle two types of package formats: (1) "source distributions", i.e. tarballs containing the raw source code and installation / build scripts, and (2) "[wheels](https://www.python.org/dev/peps/pep-0427/)", i.e. pre-build Python packages. While wheels are becoming more common for popular Python packages thanks to the efforts of projects like [manylinux](https://github.com/pypa/manylinux), they are still not as widely adapted (and as easy to build) as they should be. They are also rather rare for Windows, which leaves Windows users in the awkward position of having to build, i.e. compile, binary components within Python packages themselves.

All of the described problems - from complicated to use setuptools to the lack of pre-build wheels for Windows - let to the advent of [Python distributions](https://wiki.python.org/moin/PythonDistributions). Very similar to Linux distributions, Python distributions usually bundle pre-build Python packages with relevant toolchains and their own package managers. As of 2019, [Anaconda](https://anaconda.org/) is probably [the most widely used Python distribution](https://www.jetbrains.com/lp/python-developers-survey-2019/#PythonVersions). Its package manager is named [`conda`](https://github.com/conda/conda). Currently, Anaconda is the best if not the only option of installing a lot of relevant scientific and geospatial Python packages on Windows. Anaconda is also offered for Linux and OS X. Compared to `pip`, `conda` can manage not only Python packages but all sorts of other tools and dependencies, which makes it rather similar to [`apt`](https://en.wikipedia.org/wiki/APT_(software)), [`yum`](https://en.wikipedia.org/wiki/Yum_(software)) or [`zypper`](https://en.wikipedia.org/wiki/ZYpp). Ignoring features specific to Python, Anaconda is therefore very similar to a Linux distribution. It ships an entire stack of tools and libraries all the way down to [libc](https://en.wikipedia.org/wiki/C_standard_library) but, other than Linux distributions, no operating system kernel. Compared to Linux distributions, Anaconda also ships a much larger and significantly younger selection of Python packages. While Anaconda is technically both a company and a commercial product, the package manager itself, the package format specification and the package repository specification are open source. The [conda-forge project](https://conda-forge.org/) is probably the most widely known pure open source non-profit user of those specifications and offers independent, Anaconda-compatible [packages](https://anaconda.org/conda-forge) and a [distribution installer](https://github.com/conda-forge/miniforge). Beside conda-forge, there are also other [re-implemenations](https://github.com/QuantStack/mamba) of the `conda` package manager itself based on its specification. The conda package specification is better documented than setuptools and wheel, somewhat easier to use for complex packages and offers a much higher degree of flexibility.

## How does QGIS fit into the bigger picture?

QGIS itself has a really interesting position within the Python ecosystem. One could say that ["the relationship is complicated"](https://www.urbandictionary.com/define.php?term=It%27s%20complicated). Its massive Python API technically makes it a Python module. With sufficient background knowledge, QGIS can therefore be used in combination with virtually every other Python package in existence. The problem here is the mentioned, required background knowledge. [QGIS has been packaged as a conda-forge Python package](https://github.com/conda-forge/qgis-feedstock/), which technically makes it a Python package. While it probably has not been attempted, it could therefore also be packaged as a wheel. Besides, QGIS Python plugins are technically also [Python modules](https://docs.python.org/3/tutorial/modules.html). Because they are lacking any form of installation scripts or binary distribution formats such as wheel, they are no real Python packages. They can not (safely & reliably at least) contain binary components or simply components written in any language other than Python. However, QGIS Python plugins contain meta data in a special format, [metadata.txt](https://docs.qgis.org/3.10/en/docs/pyqgis_developer_cookbook/plugins/plugins.html#plugin-metadata), which makes them some sort of a special, feature-limited package format. QGIS Python plugin "packages" can not explicitly specify dependencies to other Python packages or non-Python tools or libraries. Strangely, QGIS (for Windows) is [shipped in a bundle](https://www.qgis.org/en/site/forusers/download.html) with a Python interpreter and a small selection of Python packages, which technically makes it a very limited Python distribution lacking any kind of reasonable package management. Also strangely, QGIS Python plugin "packages" have a very limited ability to depend on other QGIS Python plugin "packages" ("cross-plugin dependencies" or "inter-plugin dependencies") - a feature which was added to QGIS 3.8 (see [QEP #132](https://github.com/qgis/QGIS-Enhancement-Proposals/issues/132) and [PR #9619](https://github.com/qgis/QGIS/pull/9619)). Technically, this makes QGIS itself sort of a package manager with very limited dependency handling.

## Packaging Terminology

Summarizing the above, the following (at times confusing) terminology is relevant within this document:

- **Python module**: Python code sitting in a [py-file or a folder structure filled with py-files](https://docs.python.org/3/tutorial/modules.html). One Python module can import any other Python module. QGIS Python plugins happen to be Python modules. QGIS itself can be understood as a Python module, too.
- **Python package**: One or multiple Python modules with metadata and installation scripts. Source distributions and wheels are typical formats. One Python package can depend on other Python packages. QGIS Python plugins are NOT Python packages. [QGIS itself is a (conda-forge) Python package](https://anaconda.org/conda-forge/qgis).
- **QGIS Python plugin {"package"}**: [A special package format containing a single Python module](https://docs.qgis.org/3.10/en/docs/pyqgis_developer_cookbook/plugins/plugins.html). A QGIS Python plugin "package" can not (explicitly) depend on other Python packages. However, it can (with certain limitations) depend on other QGIS Python plugins. In the context of this proposal, it will sometimes be referred to as the "legacy format".
- **Non-Python components of Python packages**: E.g. actual C with build instructions or compiled C. This is what QGIS Python plugins can not contain at the moment.
- **Package manager**: A tool for installing & uninstalling packages while resolving dependency trees, among other tasks. QGIS itself has (very limited) functionality of this kind and can therefore be considered a rudimentary package manager. Otherwise, `apt`, `yum`, `zypper`, `pip` and `conda` are relevant examples.
- **Python distribution**: A usually self-contained collection of pre-built Python packages plus a package manager plus an installer. Anaconda is a typical example. The QGIS / OSGeo4W Windows installer also falls within this category.
- **Python package dependencies**: One Python package depending on another Python package, e.g. pandas depends on numpy.
- **Cross-plugin dependencies** / **inter-plugin dependencies**: One QGIS Python plugin "package" can depend on another QGIS Python plugin "package".
- Anaconda: (1) a company, (2) a product, (3) a Python distribution, (4) an open specification
- conda-forge: A non-profit project using Anaconda's open specifications and an ([almost](https://github.com/conda-forge/miniforge/issues/28)) independent Python distribution
- `pip`: A Python package manager
- `conda`: A Python package manager
- `apt`: A Linux package manager
- `yum`: A Linux package manager
- `zypper`: A Linux package manager

## Currently Missing Links

Looking at the list of relevant terminology, certain missing links stick out.

1. **QGIS Python plugins can not depend on Python packages.**
2. **Python packages can not depend on QGIS Python plugins.**

From a technical perspective, there is no good reason why both should not be possible. The root cause of the current state of affairs can be found in QGIS' history. The QGIS project established its own Python plugin "package" format and acts as a (limited) package manager independent of other Python package managers.

3. **The QGIS / OSGeo4W Windows installer does not contain a Python package manager (while in fact containing Python packages).**

In combination, all three missing links mentioned up to this point are leading to all sorts of problems and bizarr workarounds. A detailed summary can be fund in [this question](https://gis.stackexchange.com/q/196002/13332) and its answers on stackexchange.

If a plugin author wants to make a QGIS Python plugin depend on a Python package right now, there are the following options:

- The desired Python package is part of the QGIS / OSGeo4W installer - OK
- The desired Python package is NOT part of the QGIS / OSGeo4W installer:
    - Do not use the desired Python package as a dependency at all: The QGIS Python plugin might become infeasible or overly complicated.
    - Copy the desired Python package into the source tree of the QGIS Python plugin (official recommendation of the QGIS community): Hard to maintain, leading to bloat, causing licensing issues, not always possible.
    - Contacting the maintainers and asking for the inclusion of the desired package in *future* releases of QGIS / OSGeo4W (unofficial recommendation of the QGIS community): The QGIS Python plugin can not be shipped to existing QGIS / OSGeo4W installations. QGIS users need to re-install QGIS / OSGeo4W if they want to use the QGIS Python plugin.
    - Asking the users to install the desired Python package manually: Not many users can do it, even fewer are willing to do so. The process is both annoying and non-trivial. It requires to make `pip` work first.
    - Building some kind of Python package install functionality into the QGIS Python plugin, i.e. automating the previous approach: Non-trivial and very risky. Blowing up a user's QGIS installation in the process is a real possibility here.

4. **If the desired Python package contains non-Python components and is not available as a wheel, making a QGIS Python plugin depend on it in any reliable and relatively easy to distribute form is virtually impossible at this point.**

## Currently Unsupported Use-Cases

So far, the QGIS community treats QGIS Python plugins as a "scripting extension" of QGIS, which is falling far behind its actual potential. In many cases other than QGIS, Python serves as a "glue language" between different types of applications and libraries. QGIS should not be an exception here. A proper "connection" to Python packages and allowing non-Python components in them would enable the following currently highly desired use-cases, among others:

- Accelerating Python code through [transpiling](https://en.wikipedia.org/wiki/Source-to-source_compiler) it to C: [Cython](https://cython.readthedocs.io/en/latest/)
- Inclusion of C code: [C extensions](https://docs.python.org/3/extending/extending.html)
- Inclusion of C++ code: [swig](http://www.swig.org/), [sip](https://www.riverbankcomputing.com/software/sip) or [plain C++ Python extensions](https://docs.python.org/3/extending/extending.html)
- Access to Fortran libraries: [f2py](https://numpy.org/doc/stable/f2py/index.html)
- GPGPU: [cupy](https://cupy.chainer.org/), a CUDA-enabled numpy-drop-in-replacement, [PyCUDA](https://documen.tician.de/pycuda/) or [PyOpenCL](https://documen.tician.de/pyopencl/)
- Machine Learning, for instance in the context of satellite image classification and vectorization: [tensorflow](https://www.tensorflow.org/), [pytorch](https://pytorch.org/), [scikit-learn](https://scikit-learn.org/) or others

Besides, at the higher end of computing, it has become a common practice to [run operations on clusters](https://github.com/dask/dask-labextension#dask-jupyterlab-extension) and even super-computers from within for various types of [interactive notebooks](https://en.wikipedia.org/wiki/Notebook_interface), for instance [Jupyter](https://jupyter.org/) notebooks. The Python ecosystem offers excellent Python packages for this purpose, which run just fine from within QGIS (e.g. in a QGIS Python plugin or from the QGIS Python console):

- [MPI for Python](https://mpi4py.readthedocs.io/en/stable/), a classic in this field - usually used in conjunction with C, C++ and Fortran extensions
- [Dask](https://dask.org/), a rather young but well funded player in the field - noticeable, among other features, for scaling numpy-compatible arrays and pandas-compatible dataframes too large to fit into a single machine's memory across multiple machines' memory

There are examples in the wild of people doing similar things with QGIS, e.g. [at the Joint Research Centre (JRC)](https://ec.europa.eu/eurostat/cros/system/files/dawos18_lecture.soille.jeodpp.pdf). Unfortunately, deployments of this kind are currently prohibitively complicated for many smaller entities.

# 2. Proposed, Preferred Solution <!-- MUST -->

The QGIS plugin manager should be *extended*, allowing it to interact with both `conda` and `pip`. It should be allowed to distribute QGIS Python plugins (a) in the existing, unchanged "legacy" QGIS Python plugin "package" format, (b) as conda packages and (c) pip-installable wheels and source distributions. QGIS on its own should not be a package manager except for "legacy" QGIS Python plugin packages. It simply is not within the scope of the QGIS project. QGIS should merely interact with existing infrastructure. The cross-plugin dependency mechanism within the QGIS Python plugin "package" format should be cleaned up.

## QGIS Plugin Management Terminology

Concerning Python plugins, QGIS can do the following things at the moment:

- **install**: A ZIP-file containing a Python module and a `metadata.txt` file is unpacked into a new folder underneath the QGIS Python plugins folder. Other QGIS Python plugins (cross-plugin dependencies) may be installed in the process, too. In the current codebase, installing a QGIS Python plugin automatically triggers a hard-coded load-start sequence, i.e. QGIS Python plugins can not be silently installed in the background.
- **uninstall**: A QGIS Python plugin's folder underneath the QGIS Python plugins folder is removed, i.e. deleted.
- **load**: Loading a QGIS Python plugin is roughly equivalent to importing it as a Python module. Very little to no plugin code is executed at this stage.
- **unload**: Unloading a QGIS Python plugin is a procedure that tries to remove the plugin from memory by deleting all references to it so it is left to [Python's garbage collector](https://docs.python.org/3/library/gc.html). Python does not intentionally allow this use-case for imported modules, so QGIS' implementation can be considered a hack and is unreliable. However, the added benefit from this feature is the [ability to re-load a plugin at QGIS runtime](https://plugins.qgis.org/plugins/plugin_reloader/), which greatly helps with the development of QGIS Python plugins.
- **start**: Once a QGIS Python plugin is loaded, it has to be started. QGIS therefore calls the plugin's `classFactory` and/or `serverClassFactory` functions. Both of those functions return a "plugin object" with a certain API. Plugin objects originating from a `classFactory` may have an `initGui` method, which is subsequently called. A plugin object originating from `classFactory` may also have an `initProcessing` method, which may eventually be called from the `QgsProcessingExec` C++ infrastructure.
- **stop**: Before it can be attempted to unload a QGIS Python plugin, it has to be "stopped" - i.e. the plugin must be given the opportunity to run cleanup code. Plugin objects must therefore expose an `unload` method which is called prior to unloading the plugin. Technically, *stoping* and *unloading* happen in the same place within the current codebase, but they should - for the purpose of this discussion - be understood as two separate, distinct steps. While there is no clear specification, most plugins implicitly expect to be truly unloaded (i.e. removed from memory) before they can be re-started. Failing to do so results in non-deterministic behavior. Plugin objects originating from `serverClassFactory` can expose an `unload` method, but it will not be called by QGIS.

## Analysis of Current Implementation

This section is based on the codebase of QGIS 3.12.

### Tasks

QGIS' plugin management is hybrid. Compared to typical package managers such as `pip`, `conda`, `apt`, `yum` and `zypper`, QGIS is not just installing and uninstalling Python plugins. It also loads and starts as well as stops and unloads them.

### Structural Overview

The current implementation is an interesting mix of C++ and Python code. Even the plugin management GUI itself is partially written in C++ (the main window, `QgsPluginManager[Interface]`, and part of its logic) and partially written in Python (all further dialogs and their logic).

Underneath [`/python/pyplugin_installer/`](https://github.com/qgis/QGIS/tree/final-3_12_3/python/pyplugin_installer), the class [`QgsPluginInstaller`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/pyplugin_installer/installer.py#L60) from [`installer.py`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/pyplugin_installer/installer.py) is a Python API that is called from the C++-interface ([`QgsPluginManagerInterface`](https://github.com/qgis/QGIS/blob/final-3_12_3/src/app/pluginmanager/qgsapppluginmanagerinterface.cpp#L22)) through [`QgsPythonRunner`](https://github.com/qgis/QGIS/blob/final-3_12_3/src/core/qgspythonrunner.cpp#L18). Most of the relevant C++ code is located underneath [`/src/app/pluginmanager/`](https://github.com/qgis/QGIS/tree/final-3_12_3/src/app/pluginmanager) (with a single ui-file elsewhere, [`/src/ui/qgspluginmanagerbase.ui`](https://github.com/qgis/QGIS/blob/final-3_12_3/src/ui/qgspluginmanagerbase.ui)). The code in [`/python/pyplugin_installer/`](https://github.com/qgis/QGIS/tree/final-3_12_3/python/pyplugin_installer) handles plugin fetching, installation and removal, metadata processing, repositories and dependencies.

[`/src/app/qgspluginregistry.cpp`](https://github.com/qgis/QGIS/blob/final-3_12_3/src/app/qgspluginregistry.cpp) offers a class named [`QgsPluginRegistry`](https://github.com/qgis/QGIS/blob/final-3_12_3/src/app/qgspluginregistry.cpp#L48) (used by [`QgsPluginManager`](https://github.com/qgis/QGIS/blob/final-3_12_3/src/app/pluginmanager/qgspluginmanager.cpp#L64)) which is not exposed to Python at this point. `QgsPluginRegistry` makes heavy use of [`mPythonUtils`](https://github.com/qgis/QGIS/blob/final-3_12_3/src/app/qgspluginregistry.cpp#L63), which is a C++ wrapper around [`/python/utils.py`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/utils.py) (through [`/src/python/qgspythonutilsimpl.cpp`](https://github.com/qgis/QGIS/blob/final-3_12_3/src/python/qgspythonutilsimpl.cpp)). `QgsPluginRegistry` offers two relevant methods: [`loadCppPlugin`](https://github.com/qgis/QGIS/blob/final-3_12_3/src/app/qgspluginregistry.cpp#L325) and [`unloadCppPlugin`](https://github.com/qgis/QGIS/blob/final-3_12_3/src/app/qgspluginregistry.cpp#L452). It is the heart of QGIS' original, dated plugin system. At this point, it is [partially responsible for loading Python plugins](https://github.com/qgis/QGIS/blob/final-3_12_3/src/app/qgspluginregistry.cpp#L283). It is exclusively responsible for loading C++ plugins.

[`/python/utils.py`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/utils.py) can be understood as an exposed Python API (`qgis.utils`). It is called both by the QGIS C++ side and by Python plugins. Here, the [`iface`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/utils.py#L212) and [`serverIface`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/utils.py#L611) objects plus a lot of closely related infrastructure reside. Besides, this module is responsible for holding information and handles on loaded Python plugins as well as installed Python plugin inventory. It furthermore loads, unloads, starts and stops Python plugins. Completely independently of Python plugins, it contains mechanisms for QGIS Python Macros. Most noticeable, `qgis.utils` includes an [`_import`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/utils.py#L730) function, which is used to override the Python interpreter's [`builtins.__import__`](https://docs.python.org/3/library/functions.html#__import__) function. Therefore, every import statement in Python within QGIS is running through the mentioned custom `_import` function. Via this mechanism, QGIS prohibits the import of PyQt4 and tracks the import of Python plugin modules, which in return is the foundation for the plugin unloading and reloading mechanism.

[`/tests/src/app/testqgisapppython.cpp`](https://github.com/qgis/QGIS/blob/final-3_12_3/tests/src/app/testqgisapppython.cpp) is used to test the C++ to Python infrastructure (i.e. calls into `qgis.utils`). [`/tests/src/python/test_qgsserver_plugins.py`](https://github.com/qgis/QGIS/blob/final-3_12_3/tests/src/python/test_qgsserver_plugins.py) tests the QGIS server Python plugin infrastructure. [`/tests/src/python/test_plugindependencies.py`](https://github.com/qgis/QGIS/blob/final-3_12_3/tests/src/python/test_plugindependencies.py) looks at the current Python cross-plugin dependency mechanism. All other components, i.e. most of the code in `/python/pyplugin_installer/`, are largely untested.

The rather complicated and fragile combination of C++ and Python [was chosen because of the optional nature of Python within QGIS](https://lists.osgeo.org/pipermail/qgis-developer/2020-March/060523.html). If QGIS is built without Python support, the C++ plugin manager GUI remains but lacks all components relevant for Python plugins. Then, only C++ plugins can be loaded and unloaded.

### State of the Code

One aspect that immediately becomes obvious when reading QGIS' Python code is the fact that there are unnecessary relics of an automatic `2to3` conversion all over the place. This tells a lot about the state of the code already. [`2to3`](https://docs.python.org/3/library/2to3.html) is an extremely useful tool, but even under the best of circumstances, its result requires intense manual editing which appears to be missing. There are also "hidden" remains of pure Python 2 code within the codebase in truly critical places (e.g. [the module import infrastructure](https://github.com/qgis/QGIS/blob/final-3_12_3/python/utils.py#L724)), which were never properly removed although it has been impossible to build QGIS with Python 2 for many years.

Both the code underneath `/python/pyplugin_installer/` and the code in `/python/utils.py` heavily uses the `global` keyword. The overall design of the code heavily relies on global variables so it is not trivial to get rid of them. The end result can only be described as dangerous, fragile and extremely hard to debug. Static code analysis tools fail to analyze this logic properly. Global variables are even used for error handling (e.g. inside the [installer data code](https://github.com/qgis/QGIS/blob/final-3_12_3/python/pyplugin_installer/installer_data.py#L568)), which can only be described as bizarr given the surface of attack which they are offering themselves. As a direct consequence of the described use of global variables, the object-oriented nature of much of the code in `/python/pyplugin_installer/` breaks and many mission-critical classes should never ever be instantiated more than once.

Most of the Python code cares very little about [mutable Python data types](https://docs.python.org/3/reference/datamodel.html?highlight=mutable). A few calls to `copy` methods in interesting places tell a story of running into issues related to mutability without caring too much about it. If a potential user relying on the correct use of mutability would call some of the Python APIs in `qgis.utils` and `qgis.pyplugin_installer`, it would directly trigger layers of bugs.

As far as style goes (completely ignoring the more controversial sections of [PEP 8](https://www.python.org/dev/peps/pep-0008)), it is really hard to see what internal and what external APIs are, i.e. which methods are only called from within classes/modules or from the outside. Because there is no concept for public and private entities in Python, the typical convention is to use (single) leading underscores in names for hinting at "privacy". The lack of such hints is a minor detail but one that leads to all sorts of issues when maintaining code in the longer run. It makes it extremely hard to understand and follow the logic as it stands. A somewhat bigger style issue is the fact that there is almost no exception raising/handling as well as no parameter type, bounds and consistency checking in functions/methods. As long as nobody touches this code, it will not become an issue. It is only hitting in the long run as a lot of smaller pull requests fixing related issues clearly show.

Much of the code's design is based on Python lists and dictionaries. Technically, this is not a bad idea. However, based on the degree to which the lists and dictionaries are nested (up to 7 layers in some places), it is hard to follow the logic. Many functions and methods are working on multiple levels of nesting simultaneously, which makes it virtually impossible to track changes in the tree. It also makes it impossible to modify the functionality in a meaningful way without breaking stuff all over the place. On those grounds alone, the code could be considered virtually unmaintainable. It can be assumed that a lot of the redundant functionality within the plugin manager's code is a direct consequence of its lack of maintainability. Whenever new features were added, a significant portion of new redundancy was introduced in addition as many of the related pull requests of past years clearly show.

On a somewhat higher level, the plugin manager's code tries to use object orientation. Ignoring the limitations imposed by the use of global variables, it is also lacking a meaningful structure. For instance, there are two classes, [`Repositories`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/pyplugin_installer/installer_data.py#L158) and [`Plugins`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/pyplugin_installer/installer_data.py#L501), which represent *all* of the active repositories and *all* of the present plugins in one single structure each, [`qgis.pyplugin_installer.installer_data.repositories`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/pyplugin_installer/installer_data.py#L821) and [`qgis.pyplugin_installer.installer_data.plugins`](https://github.com/qgis/QGIS/blob/final-3_12_3/python/pyplugin_installer/installer_data.py#L822). A reasonable problem decomposition would lead to two classes representing one individual repository and one individual plugin each. As a consequence, the plugin manager's code technically does not exploit object orientation at all at the moment - although package management is a textbook use-case for object orientation.

While the design decisions that led to the interesting mix of C++ and Python can be understood, the final result can only be described as convoluted and prone to problems. There are code paths originating in C++ which are calling Python code which is then again calling C++ (and vice versa). This can make sense in some cases, but it certainly does not make sense in the plugin installer most of the time. Again, it makes the code hard to maintain.

In QGIS, C++ APIs are exposed to Python through [SIP](https://www.riverbankcomputing.com/software/sip). SIP is also the foundation of [PyQt](https://www.riverbankcomputing.com/software/pyqt/). [SIP is not without its issues](https://lists.osgeo.org/pipermail/qgis-developer/2020-March/060542.html). Although the Python standard library offers a lot of relevant functionality, the plugin manager's Python code heavily calls into PyQt and QGIS APIs for this exact kind of functionality, therefore virtually maximizing the exposure to SIP with all of its associated problems. The use of `QtXml` and Qt's network stack are good examples.

The metadata format `metadata.txt` is a core feature of the QGIS Python plugin "package" format. Ironically, there is not a single place that documents all of its fields or its exact, complete specification. The latter can only be derived by reading the entire plugin manager's code. More worryingly, multiple redundant and competing approaches of parsing and interpreting the metadata exist within QGIS (and [QGIS-Django](https://github.com/qgis/QGIS-Django/) for that matter). It is almost a side note that several metadata fields go by multiple different names throughout the codebase. While it should be clear to most users that they should not run plugins from untrusted sources, it is still impressive that it is theoretically possible to crash and even exploit QGIS through `metadata.txt` files alone.

Tapping as deep into the Python interpreter as QGIS does is not without its risks. The altered Python import mechanism for instance is completely untested and actually [contains bugs with respect to mutability](https://docs.python-guide.org/writing/gotchas/#mutable-default-arguments). While individual bugs of this kind can easily be fixed, it shows that not only tests but also a test strategy for this infrastructure is missing.

The plugin installation procedure is not robust. It actually directly unpacks a potential plugin ZIP-file into QGIS' Python plugins folder. Many potential errors in this process are neither caught nor handled and might leave the Python plugins folder in an unclean state.

The current plugin installation mechanism technically does not allow to install specific versions of a plugin unless the user downloads its ZIP-file manually. If the update of a plugin breaks certain functionality, a downgrade to a specific version therefore is not trivial. Similarly, an upgrade to a specific version which is not the latest version of a plugin is also non-trivial.

The [cross-plugin dependency mechanism is not thought-through](https://lists.osgeo.org/pipermail/qgis-developer/2020-April/061056.html) and has [serious design limitations](https://github.com/qgis/QGIS/pull/9619#issuecomment-476508417). The plugin installer does not look for version conflicts and does not resolve dependency trees. The choice of installing a dependency is left to the user. Simple mistakes in the specification of cross-plugin dependencies in a `metadata.txt` file can cause infinite loops in QGIS' GUI. Dependencies are only attempted to be installed after the actual plugin (which specified those dependencies) has been installed. Should the installation of dependencies fail or be rejected by the user, it might leave a broken plugin within the QGIS configuration, which can potentially cause all sorts of errors. Besides, a plugin (A) depending on another plugin (B) can not rely on plugin (B) to be loaded and started before plugin (A) is loaded and started. This is effectively prohibiting many relevant use-cases for cross-plugin dependencies. The author of plugin (A) has [a few undocumented and risky options of working around those shortcomings](https://lists.osgeo.org/pipermail/qgis-developer/2020-April/061098.html), but this really is nowhere near ideal. Strangely, there is also no real documentation on how to properly use cross-plugin dependencies or how they are supposed to actually work. Some rather unspecific pointers can only be found in [the pull request for this feature](https://github.com/qgis/QGIS/pull/9619). It can be concluded that there has never been a real analysis of actual needs and use-cases prior to implementing this feature.

### Summary of Findings

Technically, it is always a good idea to build on top of existing infrastructure and improve existing code. A decision to rewrite code (instead of attempting to refactor it) should never be taken lightly. However, given the described state of the code of the plugin manager, it is concluded that most of it is not salvageable. **The plugin manager must be re-designed and re-written from scratch before adding new features.**

## Fundamental Design Constraints for a Potential Solution

The Python integration into QGIS has been controversial. To this date, QGIS can be built without Python i.e. the Python integration is optional. This work will *not* change the status quo.

QGIS currently compiles with Python 3. However, [a minimum minor version appears to be unspecified](https://lists.osgeo.org/pipermail/qgis-developer/2020-March/060495.html). The Python ecosystem currently supports [Python 3.5](https://www.python.org/dev/peps/pep-0478/) and greater with [Python 3.5 reaching End of Life on 2020-09-13](https://devguide.python.org/#status-of-python-branches). It is therefore planned to make this work require at least [Python 3.6](https://docs.python.org/3/whatsnew/3.6.html). This particular version introduced [literal string interpolation](https://www.python.org/dev/peps/pep-0498/) and [type hints](https://www.python.org/dev/peps/pep-0484/), among other features. The use of type hints would enable the use of static code analysis tools such as [mypy](http://mypy-lang.org/) within the QGIS codebase and could, as a "side effect", greatly increase the overall code quality. Given the scale of the proposed work and the intention of keeping things maintainable, it is planned exploit type hints in this work.

None of the technical limitations and shortcomings which are motivating this proposal are the Python plugins' authors' faults. Changing APIs and/or package formats will always cause significant and undesirable disruptions. Understandably, not all plugin authors will adopt them within a reasonable time-frame - or adapt them at all. The Mozilla Foundation's changes to both Firefox and Thunderbird APIs are good examples of how this can go terribly wrong and destroy thriving plugin ecosystems. In the context of this proposal, **full backward compatibility must be maintained.** This applies to both the "legacy" QGIS Python plugin "package" format as well as every touched QGIS Python API. If QGIS interacts with `conda` and `pip`, this must work side-by-side with "legacy" QGIS Python plugin "packages". New plugin management features must coexist with existing plugin management features and must not break any of them.

Established methods of distributing QGIS must not break, i.e. the well-known OSGeo4W installer should work and behave as before. It is suggested to establish an alternative installer for Windows (maybe also for other platforms) as a consequence of this work in a subsequent project - for instance based on [miniforge](https://github.com/conda-forge/miniforge/).

The plugin manager manages Python code. It naturally heavily interacts with Python libraries and the Python interpreter for tasks like introspection and error handling. It is therefore proposed to implement this project in almost pure Python. As a side effect, this would limit the interface to C++ code, which would in return limit the overall complexity of this work considerably. Convoluted stacks of function calls from C++ into Python and then back into C++ or vice versa (like in QGIS' current plugin manager codebase) must be banned as far as possible.

`pip` does not have an official API, see [its user guidelines](https://pip.pypa.io/en/latest/user_guide/#using-pip-from-your-program) and [the unofficial API project](https://github.com/di/pip-api). Avoiding the unofficial API, the best option is to run `pip` as a [sub-process](https://docs.python.org/3/library/subprocess.html). While this approach is not unheard of, it is risky by definition and requires high code quality and thoughtful error handling. `pip` offers machine-readable output in most critical places.

In contrast, [`conda` does have an official API](https://docs.conda.io/projects/conda/en/latest/api/). Unfortunately, it does not expose every relevant feature of the command line tool. The documentation is mostly outdated and, as it appears, incomplete. Similar to `pip`, the best approach is to run the command line tool as a sub-process. `conda` offers machine-readable output in all critical places.

As of May 2020, only `conda` offers a ["dry-run" option](https://docs.conda.io/projects/conda/en/latest/commands/install.html#Output,%20Prompt,%20and%20Flow%20Control%20Options). A similar feature [has been considered for `pip`](https://github.com/pypa/pip/issues/53) but is not implemented yet. However, there are [known workarounds](https://stackoverflow.com/q/29531094/1672565).

While `pip` and `conda` are the most widely used Python package managers today, they are not the only ones. It is also fairly easy to imagine that new projects in this field are going to emerge given the massive scale of the Python ecosystem. Any implementation within QGIS should therefore be modular. If `pip` or `conda` or both become irrelevant, it should be easy to deactivate and/or drop the corresponding code. If a new package manager becomes relevant, it should be easy to add support for it. After all, Package management should not be QGIS' responsibility. Python package management is a complicated problem on its own. QGIS should merely provide a thin layer on top of third-party solutions.

Any future plugin manager code should be significantly easier to maintain and extend than the current code and naturally of much higher quality.

## Implementation Details

### Step 1: Fully backward-compatible re-implementation of the plugin management system

This work focuses on the Python side of QGIS and Python plugins. If QGIS is built without Python support, everything described in this proposal is becoming irrelevant anyway. Because of this and the proposed intense interaction with Python code in QGIS Python plugins, it is proposed to carry out a clear majority of this work in Python. It is also intended to minimize any interaction between the C++ side and the Python side of QGIS, i.e. reducing complexity and eliminating problems associated with SIP. As a consequence, a lot of existing C++ code related to plugin management is becoming obsolete can be removed. The following new structure is proposed instead:

- A new fallback plugin manager GUI written in C++. It allows to list, activate and deactivate C++ plugins - not more, not less. This GUI only becomes visible (and is only built) if QGIS is built without Python support. The fall-back GUI does not contain any reference to Python code whatsoever, i.e. no C++ code that is conditionally built if Python is present.
- A new primary plugin manager GUI written entirely in Python that replaces the C++ fall-back GUI if QGIS is built with Python support. It also allows to list, activate and deactivate C++ plugins - based on the same API the fallback GUI will be using. Beyond that, the new primary GUI will allow the management of Python plugins and Python plugin repositories.

Both, the new fallback GUI as well as the new primary plugin manager GUI should look and behave as before so users should not notice much of a difference (if any difference at all).

The old `qgis.pyplugin_installer` module will be removed entirely and substituted by a new `qgis.pluginmanager` module. The latter will hold almost all of the plugin-related infrastructure. Most of the logic in `qgis.utils` will be replaced with thin wrappers around logic in the new `pluginmanager` module. `qgis.utils` will therefore remain API-compatible from a user's perspective but loose most of its current complexity and design issues. Plugin-related APIs in `qgis.utils` can optionally be marked as deprecated and their use can be discouraged in favor of a new, clean API in `qgis.pluginmanager`.

In this first step, `qgis.pluginmanager` should offer all the features the current Python plugin mechanism has - plus a cleaner and more reliable dependency resolution, installation and loading mechanism. Only now, the plugin manager can be extended in a second step. [Most of the groundwork and analysis for the first step](https://github.com/qgist/pluginmanager) has already been done and can be found on Github.

Because backward compatibility is a major concern, the first step will technically not change the behavior of QGIS with respect to Python plugins except for the dependency mechanism. As of now, only one plugin in the public QGIS plugin repository is using this feature at all, but not in a meaningful way, so the expected problems are minimal.

### Step 2: Adding support for Python package managers

This step adds support for `conda` and `pip`. Due to the lack of proper APIs in both cases, an extremely robust [sub-process](https://docs.python.org/3/library/subprocess.html) layer will be implemented around both tools. Although this is not a trivial task, it is a common procedure in Python-based testing frameworks for non-Python software. There are [good and robust examples to draw inspiration from](https://github.com/pleiszenburg/loggedfs-python/blob/v0.0.5/tests/lib/procio.py). In a broader sense, there are also established GUI package management tools following this approach.

With the ability to use `conda` and `pip`, the next is question is how QGIS Python plugins should look like in terms of packaging. As explained earlier in this document, QGIS Python plugins are simply single Python modules with an additional `metadata.txt` file within their root directory. Other than a regular Python module, a QGIS Python plugin module can not consist of a single file only (e.g. `somemodule.py`) but always requires a directory (e.g. `somemodule/`) and an `__init__.py` file inside. QGIS Python plugins are therefore - strictly speaking - always [regular packages](https://docs.python.org/3/reference/import.html#regular-packages). Having said that, the problem is easy to solve: QGIS Python plugins can be packaged as normal conda packages or wheels for pip with three noticeable, though insignificant characteristics:

- A Python package containing a QGIS Python plugin must only contain exactly one Python module, i.e. the QGIS Python plugin module.
- A `metadata.txt` file must be present inside the root of the Python module folder. Its specification does not change for the most part, except ...
- ... because the Python package metadata (and its package manager, i.e. `conda` or `pip`) is now handling dependencies, the QGIS Python plugin `metadata.txt` file must not contain information on dependencies of any kind. Strictly speaking, it can contain this kind of information, but it will simply be ignored.

The described approach does not introduce any changes to the QGIS "legacy" Python plugin "package" format. It is therefore perfectly feasible to distribute a QGIS Python plugin in multiple formats, e.g. in the legacy format and as a conda package, while maintaining the entire project in a simple directory tree within a git repository or similar.

Because the support for `conda` and `pip` should not replace the existing "legacy" QGIS plugin "package" format, all three distribution paths are supposed to coexist. If `conda` is present on a system, it is - very simply put - usually aware of what `pip` does (while `pip` does not see what `conda` does). [Both tools behave surprisingly well next to each other](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-pkgs.html#installing-non-conda-packages) although there are well-known limitations. The interesting question then is the interaction with QGIS' legacy infrastructure. It is proposed to design the new `pluginmanager` module in a modular way, containing one "backend" each for `conda`, `pip` and the "legacy" QGIS Python plugin "package" format. The backends have a common API, so new backends can easily be added. Each backend is *on its own* responsible for installing & uninstalling QGIS Python plugins as well as handling their dependencies. Dependencies can intentionally not be handled across multiple backends, which would blow up the complexity of the implementation beyond a manageable point. The `pluginmanager` will only handle direct conflicts between the different backends and prohibit broken installations by exploiting dry-run capabilities of all involved tools. If a plugin is available from multiple backends (and/or in multiple versions), a user will be asked to choose a backend (and a version) for installation.

QGIS already supports package repositories. The concept will simply be extended by allowing `conda` and `pip` repositories with configuration options specific to those backends. In the case of `conda`, different conda repositories within a QGIS configuration could refer to different [conda package channels](https://docs.conda.io/projects/conda/en/latest/user-guide/concepts/channels.html). In the case of `pip`, different pip repositories could refer to [specific package sources](https://packaging.python.org/guides/hosting-your-own-index/).

Adding `conda` and `pip` support also raises the question of how QGIS finds and loads plugins. While both Python package managers can install Python packages into custom locations, it does not make sense even in the case of QGIS. `conda` and `pip` should therefore install all Packages into their default locations. QGIS will instead go through all paths in Python's `sys.path` and try to locate folders containing both an `__init__.py` file and a `metadata.txt` file. Those Python modules will then be considered QGIS Python plugins. There is no need for QGIS to interfere with `conda`'s and `pip`'s default configuration.

Because QGIS still allows C++ plugins, they will be collected into a special C++ plugin backend. For consistency, this backend will be exposed to a user as a single, protected plugin repository. The user will not be able to add or remove C++ repositories. The C++ backend naturally can not install or remove C++ plugins, but it should be able to activate and deactivate C++ plugins by interacting with `QgsPluginRegistry` on the C++ side of QGIS. `QgsPluginRegistry` therefore must become part of QGIS' Python API.

### Code Structure

This section proposes a rough structure for the `pluginmanager` module in the form of sub-modules.

- `qgis_api`: Collects references to the QGIS API in central location
- `error`: Special Python exception types related to plugin management
- `settings`: Additional, required infrastructure around `QgsSettings`. Because of [backward compatibility](https://lists.osgeo.org/pipermail/qgis-developer/2020-April/060790.html), this is not as straightforward as it should be.
- `imports`: Isolation of the Python module import layer (i.e. the override of `builtins.__import__`) and therefore all code related to loading and unloading Python modules
- `index`: Contains a single class, the `Index`. It is the highest level of abstraction for QGIS Python plugin management. A package index holds package repositories and individual plugins. It offers the index API, which is the layer through which the rest of QGIS and the GUI interacts with the package manager module.
- `plugin`: Contains a single class, the `Plugin`. It represents a single plugin, installed or uninstalled, regardless of the backend. It collects references to all possible, available releases of this plugin. Plugins can be protected if they are core QGIS Python plugins or C++ plugins.
- `pluginrelease`: Contains a single class, the `PluginRelease`. A plugin release represents one specific version of a plugin from one specific backend. If two backends offer the same version of a plugin, those two options of getting this version of the plugin will be considered different releases.
- `repository`: Contains a single class, the `Repository`. A plugin repository represents all plugin releases offered by one specific package source from one specific package backend. A single default C++ repository and the default public QGIS Python plugin repository will be protected.
- `backends`: Likely a folder with further sub-modules, one per backend. It contains code specific to individual backends.
- `version`: Contains a single class, the `Version`. It contains logic to parse, compare and export software versions. This will be very similar to [semver](https://github.com/python-semver/python-semver), but specific to QGIS' needs and fully backward compatible with the old version comparison logic.
- `metadata`: Contains a single class, the `Metadata`. This class represents the metadata of a single plugin release. It allows to import metadata from different sources and formats, to export metadata to different formats and to compare metadata.
- `metadatafield`: Contains a single class, the `MetadataField`. It represents one single field of metadata and handles import/parsing, comparison and serialization of the data in this field.
- `metadataspec`: Contains a list of dictionaries specifying the metadata format as well as required serialization and deserialization methods. Metadata fields derive their logic from this specification.
- `abc`: Holds [abstract bases classes](https://docs.python.org/3/library/abc.html) for type checks, among other uses.
- `core`: Initializes the package index, triggers the load and start procedures of plugins when QGIS is launched and takes care of launching the plugin manager GUI if a user requests it.
- `gui`: Likely a folder with further sub-modules containing the plugin manager GUI. It should only contain GUI code and event handling. It calls the index API for all actions related to actual package management.
- `cli`: Based on the proposed design, specifically the index API, a CLI-type alternative front-end becomes theoretically possible. This could be interesting for the QGIS Server. Its implementation is not part of this proposal, although the infrastructure will be prepared.

It is suggested to isolated `metadata`, `metadatafield`, `metadataspec` and `version` into a separate, new Python package. Ideally, both QGIS and QGIS-Django could then use this package as a common codebase for QGIS Python plugin metadata handling. Furthermore, relevant QGIS documentation could automatically be derived and updated from `metadataspec`.

A work-in-progress proof-of-concept QGIS plugin manager *plugin*, which is already using the proposed structure, [can be found here](https://github.com/qgist/pluginmanager).

## Affected APIs

**From a user's perspective, no QGIS API will change.** From a QGIS developer's perspective, there will be internal changes. This section lists those internal changes.

In terms of Python, the following APIs will be changed:

- `qgis.pyplugin_installer`: This module will be removed (an substituted) entirely by `qgis.pluginmanager`. `pyplugin_installer` was very likely never meant to be a public API, so this is not expected to cause any disruption.

In terms of C++, the following classes will be changed:

- `QgsPluginRegistry`: Simplification and cleanup
- `QgsPythonUtilsImpl`: Simplification and cleanup
- `QgsPluginManager`, `QgsAppPluginManagerInterface` and closely related classes: Massive removal of code and functionality.

## Affected Files <!-- MUST -->

Relative to the root of the QGIS 3.12 codebase:

- `/python/utils.py`: Although some plugins are using APIs from this module, [it is unclear which methods are meant to be stable API](https://lists.osgeo.org/pipermail/qgis-developer/2020-April/061056.html) and which methods and properties are internal. While its APIs will therefore be kept completely compatible, about 90% of its code will be rewritten.
- `/python/pyplugin_installer/*.py`: While from a user's perspective the UI will be kept unchanged, this this code will be rewritten entirely and the mentioned files removed. Because this is not a public API, compatibility is not a concern here.
- `/python/pyplugin_installer/*.ui`: Can remain largely untouched but will be moved to new `pluginmanager` module folder.
- `/src/ui/qgspluginmanagerbase.ui`: Will be moved into the `pluginmanager` module folder.
- `/src/app/pluginmanager/*.*`: Most of the current logic will be rewritten in Python and therefore removed here. What remains is a skeleton fall-back UI that allows to activate & deactivate C++ plugins if QGIS is built without Python.
- `/src/app/qgspluginregistry.*`: These files offer mostly redundant features which can be cleaned up. At the end, they should only manage C++ plugins. Its class should be exposed to the C++ fallback UI and the new Python plugin manager.
- `/src/python/qgspythonutilsimpl.cpp`: This file offers a C++ layer into `/python/utils.py`. Most of this code can be removed, basically only leaving the Python thread initialization and termination.
- `/tests/src/app/testqgisapppython.cpp`: Tests of obsolete C++ code can be removed.

## Recommended changes to related projects

The proposed work would greatly benefit from the following changes in projects related to QGIS (while strictly speaking not requiring them):

- [QGIS-Django](https://github.com/qgis/QGIS-Django), which offers `plugins.xml`, should add two new fields to it: `plugin_dependencies` and a hash sum for plugin ZIP-files.

Currently, both QGIS and QGIS-Django handle plugin metadata but maintain separate and slightly different code for parsing, analyzing and validating plugin metadata files. As part of the proposed work, it is suggested to make both projects use a common code basis for this purpose.

## Example <!-- MUST -->

[A work-in-progress proof-of-concept implementation](https://github.com/qgist/pluginmanager).

# 3. Alternative, Unfavorable Solutions

As a part of this proposal, alternative approaches, (partial) solutions and different design considerations were explored. The following list contains several noteworthy candidates and the reasons why those are unfavorable or even impossible:

1) Staying at the current solution by not implementing the proposed work: See ["Analysis of Current Implementation"](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#analysis-of-current-implementation), ["Currently Unsupported Use-Cases"](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#currently-unsupported-use-cases) and ["Currently Missing Links"](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#currently-missing-links) sections of this document.
1) Refactoring the current implementation: See the ["Summary of Findings"](https://github.com/qgist/pluginmanager-qep/blob/master/QEP.md#summary-of-findings) sub-section within the "Analysis of Current Implementation" section of this document.
1) Dropping support for the "legacy" QGIS Python plugin "package" format: It would simplify the proposed work significantly. However, breaking backward compatibility at this scale has the potential to destroy the existing plugin ecosystem.
1) Dumping the current plugin installer for good in favor of conda and/or pip: See the previous point.
1) Designing a new QGIS Python plugin package format or significantly improving the existing "legacy" QGIS Python plugin "package" format - instead of relying on conda packages and pip-installable wheels: Extremely complicated and simply too far beyond the scope of the QGIS project. Large entities working on nothing but package management solutions are massively struggling with similar tasks.
1) Implementing a real package manager as part of QGIS instead of relying on conda and pip: See the previous point.
1) The proposed work is implemented as a separate QGIS Python plugin: Partially possible, see [proof-of-concept](https://github.com/qgist/pluginmanager). The primary concern here is the fact a separate, external plugin manager would have to inject code into QGIS at run time which makes it hard to maintain and risky to operate. It also can not solve the described problems concerning plugin dependencies and the plugin loading sequence because it would be a plugin itself at the mercy of the current implementation. It would lead to a massive increase in complexity.

# 4. Performance Implications <!-- MUST -->

- **While QGIS performs tasks not related to plugin management: None.**
- While loading QGIS: Likely some, but it is hard to tell whether the process becomes faster or slower. In any case, the change will not be significant. The overall cleanup should, however, allow some optimizations which the current plugin manager code does not allow in a clean manner.
- A repository refresh is very likely going to require more time than before. Early test code suggests single digit seconds per repository per refresh. Because a refresh only happens if a user opens the plugin manager GUI, it is safe to say that it will not have any negative impact on the overall user experience. The behavior is expected to be similar to for instance running `apt update`.

# 5. Backward Compatibility <!-- MUST -->

**Full backward compatibility for plugins will be maintained.**

A minimal exception is made with respect to the current cross-plugin dependency mechanism. Due to its lack of proper specification and documentation as well as only a single plugin using it as of May 2020, no serious problems are to be expected. This plugin is going to continue to work without changes. In terms of proprietary plugins and their potential current use of cross-plugin dependencies, a survey might be conducted to better understand the actual needs of their developers.

It is suggested that QGIS can, as a consequence of this proposal, not be built with Python 3.5 or prior while building with Python can remain optional. This should not have any noticeable effect on backward compatibility as breaking changes in the Python interpreter have become extremely rare and specific (usually minor changes to the standard library) after the Python 2 to 3 transition and its associated massive pain.

It is suggested to scan the entire public QGIS plugin repository of actively maintained plugins compatible with QGIS 3 with automated tests for potential issues prior to a future QGIS release potentially containing the proposed changes. A CI-based testing infrastructure could even go as far as installing, loading and starting every single available, theoretically compatible plugin.

# 6. Copyright

This document has been placed in the public domain under the GPL, see [license](https://github.com/qgist/pluginmanager-qep/blob/master/LICENSE). The copyright belongs to [its authors](https://github.com/qgist/pluginmanager-qep/blob/master/AUTHORS.md).

# 7. Issue Tracking ID(s) <!-- MUST -->

None. <!-- TODO -->

# 8. Votes <!-- MUST -->

None. <!-- TODO -->
