\page build_documentation Building the documentation

This site is built from its own repository,
<a href="https://github.com/pockerman/bitsim_bitrl_docs">bitsim_bitrl_docs</a>, which has bitsim and bitrl
as git submodules in `libs/`. The pages of the examples live with the examples in the two libraries; the
guides, such as \ref bitsim_sensors_guide, live in `doc/` here. Every night the repository moves the two
submodules to the latest commit of each library and rebuilds the site.

## Dependencies

You need <a href="https://www.doxygen.nl/">Doxygen</a>, 1.9 or newer, and
<a href="https://www.graphviz.org/">Graphviz</a> for the diagrams:

@code{.sh}
sudo apt install doxygen graphviz
@endcode

## Build

Clone the repository with its submodules:

@code{.sh}
git clone --recurse-submodules https://github.com/pockerman/bitsim_bitrl_docs
cd bitsim_bitrl_docs
@endcode

In a clone made without `--recurse-submodules`, get them with `git submodule update --init`. Then run
Doxygen from the top of the repository:

@code{.sh}
doxygen Doxyfile
@endcode

The site is written to `docs/html`; open `docs/html/index.html` in a browser.

## The latest bitsim and bitrl

The submodules are at the commits the repository last recorded. To build the site from the latest commit
of each library instead:

@code{.sh}
git submodule update --remote libs/bitsim libs/bitrl
doxygen Doxyfile
@endcode

## What goes where

| File | What |
| ---- | ---- |
| `Doxyfile` | the Doxygen settings: the inputs, `libs/bitsim`, `libs/bitrl` and `doc/`, and the output |
| `DoxygenLayout.xml` | the menu at the top of every page |
| `doc/index.md` | the front page |
| `doc/*.md` | the guides to bitsim, and this page |
| `doc/bitrl/*.md` | the pages about bitrl |
| `libs/bitsim/examples`, `libs/bitrl/examples` | the examples and their pages, in the libraries themselves |

A new guide is a Markdown file in `doc/` whose first line is a Doxygen page command with the page's id and
title, as in the guides already there; to put it in the menu, add it to `DoxygenLayout.xml`.
