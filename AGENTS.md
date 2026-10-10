# Working on TOML

TOML is a native iyi module: a TOML 0.5.0 parser, ported from the Crystal
shard toml.cr 0.8.1. [README.md](README.md) is for using it; this page is
for changing it.

## Layout

- `toml.iyi` is the whole module, in the original's order: `TOML` (with
  `TOML.parse`, `TOML.quote` and `TOML::Any`), then the private `Token`,
  `Lexer` and `Parser`. It is one file because iyi refuses an import cycle,
  and `Parser` and `TOML::Any` need each other.
- `toml_test.iyi` runs toml.cr's specs and every case under `spec/`.
- `archive/crystal/` is the Crystal original, kept as a reference. Do not
  change it.

## Checks

```sh
iyi test
iyi build toml_test.iyi --emit-iyimod /tmp/iyimod -o /tmp/toml_test
iyi fmt --check toml.iyi toml_test.iyi
```

CI runs the first two. The second fails when an exported `def` is not fully
typed: a consumer reads the module's surface from its `.iyimod`.

## Rules

- A change of behaviour gets a case in `toml_test.iyi`, and a line in the
  README's "Differences from toml.cr 0.8.1" when it departs from the
  original.
- Errors name the position: the lexer's and the parser's `raise` add
  `TOML: … at line:column`.
- Whatever `TOML.quote` writes, `TOML.parse` reads back unchanged.
- Follow the original's code closely where nothing needs fixing; it keeps
  the port comparable to `archive/crystal/`.
