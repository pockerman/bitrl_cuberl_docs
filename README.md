# bitrl_cuberl_docs

The documentation of bitsim, the robotics library, and bitrl, the reinforcement learning
library. They are the submodules `libs/bitsim` and `libs/bitrl`.

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
