The mithril or snabbdom UI modules are in `ui/`. Say we work on `ui/round`, the playing UI.

Enable auto-recompile:

```
cd ui/round
gulp
```

Build with a custom chessground:

```
cd ui/round
rm -rf node_modules/chessground && ln -s ~/path/to/chessground node_modules
gulp
```

And enable auto-recompile in `~/path/to/chessground`:

```
cd ~/path/to/chessground
tsc -watch
```