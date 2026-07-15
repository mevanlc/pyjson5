# pyjson5

This is a fork of [dpranke/pyjson5](https://github.com/dpranke/pyjson5)
focused on producing more human-readable JSON5 when string values are long or
contain embedded newlines.

Refer to [upstream's README.md](https://github.com/dpranke/pyjson5/blob/main/README.md)
for general project information, installation and API documentation,
compatibility notes, and upstream release history.

## Motivation

JSON and JSON5 normally encode an embedded newline as `\n`, leaving the entire
string on one physical source line. Long prose and other large string values can
therefore be awkward to read and review even when the surrounding document is
nicely formatted.

JSON5 also supports a backslash immediately before a physical newline. That
line continuation contributes no character to the decoded value, which means it
can be combined with an ordinary `\n` escape or inserted elsewhere in a string
to improve the source layout without changing the string itself. This fork adds
command-line and encoder controls for doing that automatically.

## Fork-specific formatting features

### Show embedded newlines on separate source lines

`--multiline` inserts a JSON5 line continuation after every encoded newline in
a string value:

```console
$ python -m json5 --multiline --quote-style prefer_single \
    -c '{message: "the quick\nbrown fox jumps\nover the lazy dog"}'
{
    message: 'the quick\n\
brown fox jumps\n\
over the lazy dog',
}
```

The `\n` escapes still produce newline characters in the decoded value. The
backslashes at the physical line endings only continue the JSON5 string literal.

### Wrap long string values

`-C COL` / `--continuations-at COL` uses value-preserving continuations to wrap
long string values at or before the specified one-based output column when
possible:

```console
$ python -m json5 --continuations-at 28 --quote-style prefer_single \
    -c '{message: "the quick brown fox jumps over the lazy dog"}'
{
    message: 'the quick \
brown fox jumps over the \
lazy dog',
}
```

`-S` / `--continuations-style` selects the wrapping behavior:

* `w1` (the default) wraps at word boundaries and processes only strings
  without embedded newlines.
* `wn` wraps at word boundaries and also processes strings with embedded
  newlines.
* `c1` wraps between Unicode codepoints and processes only strings without
  embedded newlines.
* `cn` wraps between Unicode codepoints and also processes strings with
  embedded newlines.

When `--continuations-at` and `--multiline` are combined, width wrapping is
performed first and embedded newlines are folded afterward.

The equivalent `dump()` and `dumps()` keyword arguments are `multiline`,
`continuations_at`, and `continuations_style`. The last accepts a
`ContinuationStyle` enum value or its string form.

### JSON Lines shorthand

The upstream `--json-lines` option is also available as `-l`.

## Important formatting details

* These features process string values, not object keys.
* Continuation lines are not indented because leading whitespace would become
  part of the decoded string value.
* Encoded escape sequences are never split. The `c` styles wrap between Unicode
  codepoints, not bytes.
* An indivisible word or escape sequence may extend past the requested column.
* `--as-json` disables continuations because standard JSON does not support
  them.

## Development

This fork intentionally replaced upstream's `run` helper with a `justfile`; use
`just --list` to see the available development commands.

The development environment uses [uv](https://docs.astral.sh/uv):

```console
$ uv sync --group dev
$ just test
```

Regenerating `json5/parser.py` additionally requires a sibling checkout of
[dpranke/glop](https://github.com/dpranke/glop):

```console
$ git clone https://github.com/dpranke/glop ../glop
$ just regen
```
