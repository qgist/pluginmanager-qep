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

QGIS offers the possibility implement plugins, both in C++ and in Python. While it is complicated to implement and even harder to distribute C++ plugins, it is relatively easy to implement and distribute Python plugins. QGIS' Python APIs offer access to almost all relevant QGIS features. Therefore, QGIS extensibility through Python can be considered one of its greatest strengths.

As of May 2020, [Python is the third-most popular programming language](https://www.tiobe.com/tiobe-index/) in active use only outperformed by C and Java. In this context, Python is currently seeing a massive increase in adoption across many types of different applications and disciplines - in a lot of cases driving users and developers away from established domain-specific proprietary solutions and into open source. It is safe to say that Python did something that only very few open source projects and virtually no programming language on its own managed to do: It has diverted significant amounts of funding into the open source ecosystem. The Python package ecosystem reflects that.

Distributing Python packages currently is a non-trivial problem, unfortunately.

# Proposed Solution <!-- MUST -->

<!-- TODO -->

## Example(s) <!-- MUST -->

<!-- TODO -->

## Implementation details

<!-- TODO -->

## Affected Files <!-- MUST -->

<!-- TODO -->

# Performance Implications <!-- MUST -->

<!-- TODO -->

<!-- likely minimal increase in QGIS loading time -->
<!-- no effect at run-time -->
<!-- repo-refresh when installing plugins requires time, similar to `apt update` or `zypper refresh` -->

# Further Considerations/Improvements <!-- MUST -->

<!-- TODO -->

# Backwards Compatibility <!-- MUST -->

<!-- TODO -->

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
