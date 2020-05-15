# QGIS Enhancement Proposal: **Turning Plugin Management into Actual Package Management**

<table>
    <tr><th>date</th><td> 2020-05-24</td></tr>
    <tr><th>author</th><td><a href="http://www.pleiszenburg.de/?qep">Sebastian M. Ernst</a> (<a href="https://github.com/s-m-e">@s-m-e</a>)</td></tr>
    <tr><th>contact</th><td>ernst@pleiszenburg.de</td></tr>
    <tr><th>maintainer</th><td><a href="https://github.com/s-m-e">@s-m-e</a></td></tr>
    <tr><th>version</th><td>QGIS 3.X</td></tr> <!-- TODO -->
</table>

# Abstract <!-- MUST -->

<!-- TODO -->

# Motivation

QGIS offers the possibility to implement plugins, both in C++ and in Python. While it is complicated to implement and even harder to distribute C++ plugins, it is relatively easy to implement and distribute Python plugins. QGIS' Python APIs offer access to almost all relevant QGIS features. Therefore, QGIS's extensibility through Python can be considered one of its greatest strengths.

As of May 2020, [Python is the third-most popular programming language](https://www.tiobe.com/tiobe-index/) in active use only outperformed by C and Java. In this context, Python is currently seeing a massive increase in adoption across many different types of applications and disciplines - in a lot of cases driving users and developers away from established domain-specific proprietary solutions and into open source. It is safe to say that Python did something that only very few open source projects and virtually no programming language on its own managed to do: It has diverted significant amounts of funding into the open source ecosystem. The Python package ecosystem reflects that.

Distributing Python packages is currently a non-trivial problem, unfortunately. It can be considered the Achilles' heel of the Python ecosystem. Fundamental, mission-critical tools such as [setuptools](https://github.com/pypa/setuptools) and [distutils](https://docs.python.org/3/library/distutils.html) are infamous for their complicated use as well as their incomplete and inconsistent documentation. Younger projects like [flit](https://github.com/takluyver/flit) try to offer modern packaging tools and good documentation but are lacking much desired features. Somewhat more established projects like [cookiecutter](https://cookiecutter.readthedocs.io/) try to address known shortcomings by offering Python project template engines as well as many useful Python project templates. The latter solve some problems but, above a certain degree of complexity, still require deep background knowledge for actually being useful.

For the purpose of this discussion, Python packages can be divided into two categories: Packages without binary components and Packages with binary components. The first category is relatively easy to package and distribute across all relevant platforms. The second category is both problematic but also much more interesting. One of Python's strengths is its ability to play nice with all sorts of other programming languages, e.g. C, C++ or Fortran. It has therefore become natural to integrate components written in other programming languages into Python packages. A clear majority of contemporary scientific Python packages are following this approach and greatly benefit from it. It allows all sorts of interesting applications, such as accessing established numerical libraries, actual thread-based parallelism (something the [CPython interpreter](https://en.wikipedia.org/wiki/CPython) itself [is not capable of](https://wiki.python.org/moin/GlobalInterpreterLock) at the moment) and [access to GPUs for general-purpose computing](https://en.wikipedia.org/wiki/General-purpose_computing_on_graphics_processing_units). The downside is that both packaging *and installing* Python packages of this kind usually requires rather significant tool chains and sufficient background knowledge, among other issues. In the Linux ecosystem, the mentioned tool chains (as well as the mentioned background knowledge) are rather common, so it is safe to say that (not all but) many Linux users wont have massive problems installing common scientific Python packages. On OS X, the overall picture is somewhat similar. On Windows however, things are known to be highly problematic.

The most widely used distribution channel for Python packages happens to be the [Python Package Index](https://pypi.org/). As of May 2020, it holds roughly 1/4 million packages. Python packages can be installed from the index through [pip](https://pip.pypa.io/en/stable/), a fully featured [package manager](https://en.wikipedia.org/wiki/Package_manager). Both pip and the index can basically handle two types of package formats: So called "source distributions", i.e. tarballs containing the raw source code, and "[wheels](https://www.python.org/dev/peps/pep-0427/)", i.e. pre-build packages. While wheels are becoming more common for popular Python packages thanks to the efforts of projects like [manylinux](https://github.com/pypa/manylinux), they are still not as widely adopted (and as easy to build) as they should be. They are also rather rare for Windows, which leaves Windows users in the awkward position of having to build, i.e. compile, binary components within Python packages themselves. All of the described problems let to the advent of [Python distributions](https://wiki.python.org/moin/PythonDistributions). Very similar to Linux distributions, Python distributions bundle pre-build Python packages usually together with relevant toolchains and their own package managers. As of May 2020, [Anaconda](https://anaconda.org/) is probably the most widely used Python distribution. Its package manager is named [conda](https://github.com/conda/conda). As of May 2020, Anaconda is the best if not the only option of installing a lot of relevant scientific Python packages on Windows. While Anaconda is technically both a company and a commercial product, the package manager itself, the package format specification and the package repository specification are open source. The [conda-forge project](https://conda-forge.org/) is probably the most widely known pure open source non-profit user of those specifications and offers independent, Anaconda-compatible [packages](https://anaconda.org/conda-forge) and a [distribution installer](https://github.com/conda-forge/miniforge). Independently of conda-forge, the conda package manager itself has for instance also been [re-implemented](https://github.com/QuantStack/mamba) based on its specification.

# Proposed, Preferred Solution <!-- MUST -->

<!-- TODO -->

## Example(s) <!-- MUST -->

<!-- TODO -->

## Design Constraints

<!-- Limit to certain younger versions of Python, e.g. 3.6 -->
<!-- Maintain ability to build QGIS without Python -->
<!-- Maintain full backwards compatibility for plugins and APIs -->
<!-- New features should co-exist with old features -->
<!-- Old QGIS distribution methods should not break -->
<!-- Python first, C++ second - reduce interface code on both sides, less complex stacks -->

## Implementation Details

<!-- TODO -->

## Affected APIs

<!-- TODO -->

## Affected Files <!-- MUST -->

<!-- TODO -->

# Alternative, Unfavorable Solutions

<!-- TODO -->

<!-- (a) entirely move to new package format: breaks backwards compatibility -->
<!-- (b) implement own package manager: rather complicated, already solved by pip and conda -->
<!-- (c) dump old plugin manager in favor of either pip or conda: breaks backwards compatibility for existing plugins -->
<!-- (d) stay on current solution: no-go -->
<!-- (e) instead of re-write, refactor / cleanup of current solution: code is just too bad in terms of quality and complexity -->

# Performance Implications <!-- MUST -->

<!-- TODO -->

<!-- likely minimal increase in QGIS loading time -->
<!-- no effect at run-time -->
<!-- repo-refresh when installing plugins requires time, similar to `apt update` or `zypper refresh` -->

# Further Considerations/Improvements <!-- MUST -->

<!-- TODO -->

# Backwards Compatibility <!-- MUST -->

<!-- TODO -->
<!-- Short and simple: Full backwards compatibility for plugins themselves and API -->

# References

<!-- TODO -->

<!-- mailing list discussions etc -->
<!-- pull request on plugin dependencies and related QEP -->

# Copyright

This document has been placed in the public domain under the GPL, see [license](https://github.com/qgist/pluginmanager-qep/blob/master/LICENSE).

# Issue Tracking ID(s) <!-- MUST -->

<!-- TODO -->

# Votes <!-- MUST -->

<!-- TODO -->
