# User interface

The mithril or snabbdom UI modules are in `ui/`. Say we work on `ui/round`, the playing UI.

Enable auto-recompile:

```
cd ui/round
gulp
```

# Translations

1. New translation keys are added in `translation/source/site.xml` in British English.
1. Then regenerate translation keys for Scala: `node bin/trans-dump.js`
1. Do not touch `translations/dest/`. New translations from crowdin will automatically be applied here.