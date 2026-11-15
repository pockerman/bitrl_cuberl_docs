# bitrl-cubrl-docs

## Update the documentation

```
git submodule update --remote
git add -u
git commit -m "Your message"
```

The site is rebuilt on every push to `main`. Every night at 03:00 UTC the
`Build and Deploy Docs` workflow also moves the submodules to the latest bitrl
and cuberl, commits that to `main` and rebuilds the site if anything changed.
Start it by hand from the Actions tab to update right away.
