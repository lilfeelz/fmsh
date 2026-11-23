# fmsh

Structured shell commands from Apple's on-device Foundation Models API. Prints up to
three candidates, most likely first, one command per line. It never executes them.

```sh
$ fmsh 'find files larger than 100 MB'
find . -type f -size +100M
```

## Status

Experimental and unreliable. The on-device model often returns wrong or unsafe commands,
and the candidates are not validated. Treat `fmsh` as a prompt helper, not an oracle, and
review every command before running it.

## Requirements

macOS 26+, Apple silicon, Apple Intelligence enabled.

## Build

```sh
git clone https://github.com/lilfeelz/fmsh.git
cd fmsh
swift build -c release
.build/release/fmsh 'find files larger than 100 MB'
printf '%s\n' 'list hidden files' | .build/release/fmsh
```

## Install

```sh
mkdir -p ~/.local/bin
curl -fsSL https://github.com/lilfeelz/fmsh/releases/latest/download/fmsh -o ~/.local/bin/fmsh
chmod +x ~/.local/bin/fmsh
```

Add `~/.local/bin` to `PATH`, then invoke `fmsh` directly or pipe its output to a
command picker.

## Usage

```text
fmsh [-p prefix] [-c notes.md]... [question]
```

| Flag | Effect |
|---|---|
| `-p` | preserve a command prefix |
| `-c` | add notes, repeated, limited to 8,000 characters in the prompt |

Reads stdin when the question is omitted.

- Instructions: `Sources/fmsh/instructions.md`, embedded at build; `{{shell}}` expands to the shell name.
- Override: nonempty `FMSH_INSTRUCTIONS` (file path) wins and exits `1` if unreadable; otherwise `$XDG_CONFIG_HOME/fmsh/instructions.md` (default `~/.config`) is used when readable.
- Exit codes: `1` invalid input or file read, `2` model unavailable, `3` generation failed after one retry.
- Generation uses the on-device model, not a remote API. Candidates are not validated and may be invalid or unsafe; review before running.

## Layout

```text
Sources/fmsh/fmsh.swift        parse input, prompt model, normalize candidates
Sources/fmsh/instructions.md   embedded default model instructions
Plugins/EmbedInstructions/     Swift build-tool plugin
Scripts/embed-instructions.sh  generates the embedded Swift string
Tests/fmshTests/               parsing, prompts, normalization and ANSI tests
```

## Development

```sh
swift test
swift build -c release
swift format lint --strict --recursive Sources Tests Plugins
prek run --all-files
```

Swift 6.2+ and the macOS 26 SDK; no external Swift packages. Model availability depends
on hardware, settings and the model download.

## License

See [LICENSE](LICENSE).
