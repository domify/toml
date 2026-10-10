# TOML

A [TOML](https://toml.io/) parser for [iyi](https://iyi.dev/), compliant
with version 0.5.0 of TOML.

This is a native iyi port of [toml.cr](https://github.com/crystal-community/toml.cr),
the TOML parser for Crystal: its lexer, parser and `TOML::Any` at release
0.8.1, and its tests. The Crystal code is kept unchanged in
[`archive/crystal/`](archive/crystal/); toml.cr itself lives on at
crystal-community. Priit Tark at [domify](https://github.com/domify)
maintains the iyi port for now.

## Installation

```sh
iyi get github.com/domify/toml
```

## Usage

```iyi
import github.com/domify/toml::{TOML}

toml = TOML.parse(File.read("config.toml"))
puts toml["title"].as_s                # => "TOML Example"
puts toml["owner"]["name"].as_s        # => "Lance Uppercut"
puts toml["database"]["ports"][0].as_i # => 8001

File.write("names.toml", "name = #{TOML.quote("say \"hi\"")}\n")
```

`TOML.parse(string)` is the document as a `TOML::Table`, a
`Hash(String, TOML::Any)`. A `TOML::Any` holds a `Bool`, `Int64`, `Float64`,
`String`, `Time`, `Array(TOML::Any)` or table: `[]` and `[]?` go into an
array or a table, `as_s`, `as_i`, `as_i64`, `as_f`, `as_bool`, `as_a` and
`as_h` take the value out, and their `?` forms answer `nil` for a value of
another kind. `raw` is the value itself, and `kind` names it.

`TOML.quote(value)` is a basic string, `"..."`, that `TOML.parse` reads
back as `value`, for writing TOML.

Invalid TOML stops the program with its position, `TOML: duplicated key:
'a' at 2:5`. To go on after one, parse in a task and read its outcome; the
panic is still printed on the error stream:

```iyi
group do |g|
  outcome = g.spawn { TOML.parse(text); 0 }.value
  puts outcome.message if outcome.is_a?(Panicked)
  0
end
```

## Differences from toml.cr 0.8.1

- Input is a `String`: there is no `IO` overload and no `parse_file`, and an
  error is a panic, not a `TOML::ParseException`.
- iyi's `Time` is UTC only, so a local date-time such as
  `1979-05-27T07:32:00` is read as UTC, and a local time such as `07:32:00`
  as that time of the current UTC day.
- `TOML::Any` has `kind`, and no `dig`, `as_f32`, `dup` or `clone`; other
  values do not compare equal to a `TOML::Any`.
- `TOML.quote` is new.
- Fixed bugs of the original:
  - `\uXXXX` takes exactly four hex digits and `\UXXXXXXXX` eight; the
    original read `\u` with up to eight, so `"\u0001ab"` was one character,
    and refused `\U`.
  - A surrogate code point in `\u` is refused instead of crashing.
  - A date followed by a single space and a comment, `a = 1979-05-27 # x`,
    no longer loses the `#`.

## Development

```sh
iyi test
```

`toml_test.iyi` holds the tests of toml.cr's `spec/parser_spec.cr`,
`spec/lexer_spec.cr` and `spec/cases_spec.cr`, whose cases, from
[toml-test](https://github.com/BurntSushi/toml-test), are under `spec/`, and
the port's own additions. [AGENTS.md](AGENTS.md) is for working on the
module.

## Thanks

TOML for iyi exists because of the people who wrote toml.cr:
[Ary Borenszweig](https://github.com/asterite), who created it in 2015,
and [kojix2](https://github.com/kojix2), maiha, Will Richardson,
summer-alice, Andra Antariksa, Martin Verzilli, Brian J. Cardiff,
James Taylor, Michael Fellinger, Stygmates, Tyler Gillies,
V. Elenhaupt and vjdhama. Their history is in this repository's git log.

I maintain this port for the time being. If one of toml.cr's authors would
like to look after the iyi port, you are very welcome to take it over; open
an issue.

## License

MIT, see [LICENSE](LICENSE). toml.cr, which this is ported from and which
is kept in `archive/crystal/`, has no license file of its own.
