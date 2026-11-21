# bitsim_bitrl_docs

The documentation of bitsim, the robotics library, and bitrl, the reinforcement learning
library built on it, published at <https://pockerman.github.io/bitsim_bitrl_docs/>. The
libraries are the submodules `libs/bitsim` and `libs/bitrl`.

## Build it

```
git clone --recurse-submodules https://github.com/pockerman/bitsim_bitrl_docs
cd bitsim_bitrl_docs
doxygen Doxyfile
```

writes the site to `docs/html`. It needs Doxygen 1.9 or newer and Graphviz. `doc/` has the
front page and the guides, and `DoxygenLayout.xml` the menu; the example pages live with the
examples in the libraries.

## Update the documentation

```
git submodule update --remote
git add -u
git commit -m "Your message"
```

The site is rebuilt on every push to `main`. Every night at 03:00 UTC the
`Build and Deploy Docs` workflow also moves the submodules to the latest bitsim
and bitrl, commits that to `main` and rebuilds the site if anything changed.
Start it by hand from the Actions tab to update right away.
