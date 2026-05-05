---
name: bash-scripting
description: Guidelines for writing production-quality bash scripts. Use whenever asked to write a bash script.
license: MIT
---

<role>
You are an expert bash developer who writes clean, portable, and robust shell scripts.
</role>

<cli_design_principles>

## Human-Friendly Design

- Provide `--help` and `-h` flags; show usage on invalid input
- Use full words for long flags (`--output` not `--outp`)
- Confirm destructive actions unless `--force` is passed
- Show progress for long operations; support `--quiet` and `--verbose`

## Composability

- Write output to stdout by default but support `--output <file>`
- Write errors and diagnostics to stderr only
- Accept stdin when `-` is given as filename
- Produce one record per line for pipeline friendly text output
- Offer `--json` for machine-parseable output when appropriate

## Exit Codes

- `0` = success
- `1` = general error
- `2` = invalid usage

## Error Messages

- Format: `scriptname: error: what went wrong`
- Include what happened and how to fix it
- Suggest `--help` on invalid usage

## Debugging

- Add support for debuggin output if appropriate when a environment variable `DEBUG=1` is set.

## Robustness

- Validate arguments before doing any work
- Fail fast—don't partially complete then error
- Clean up temp files on exit (use trap)

## Usage Help

Put CLI help as comments at the top of the script following this template:

```bash
#!/usr/bin/env bash

#
# What is the script doing?
#
# USAGE
#
#    COMMAND SUB_SOMMAND --OPTION                               # help text for this command
#
# COMMANDS
#
#    SUB_COMMAND_1                                              # COMMAND_1 does this
#    SUB_COMMAND_2                                              # COMMAND_2 does that
#
# OPTIONS
#
#    -a OPRION_A                                                # OPTION_A does this
#    -b OPRION_B                                                # OPTION_B does that
#
# EXAMPLES
#
#    COMMAND -a OPTION                                          # This does this
#

set -euo pipefail

show_help() {
# shellcheck disable=SC2086
awk '/^[^ #]/{c=1}c==0{print $0}' $0 | sed -n '/^#/p' | sed 1d | sed 's/^#/ /g' |
  perl -pe "s/ #(.*)$/$(tput setaf 0)\1$(tput sgr 0)/" |
  perl -pe "s/(USAGE|EXAMPLES|COMMANDS|OPTIONS)/$(tput setaf 0)\1$(tput sgr 0)/" |
  perl -pe "s/\`(.+)\`/$(tput sgr 0 1)\1$(tput sgr 0)/"
exit 1
}

show_help
```

</cli_design_principles>

Be idempotent where possible (safe to run twice).

On commands which are changing data or critical, ask for confirmation from the command user using this script

```bash
yesno() {
  echo ""
  read -r -p " $1 [Y/n] " response
  [[ $response == "n" || $response == "N" ]] && exit 1
}

yesno "are you sure to do this?"
```

<bash_coding_standards>

## Script Header

Always start with:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

## Variables

- Lowercase for local: `local filename`
- UPPERCASE for exported/constants: `readonly VERSION="1.0.0"`
- Always quote: `"$var"` not `$var`
- Defaults: `${var:-default}`, required: `${var:?error message}`

## Conditionals

- Use `[[ ]]` not `[ ]`
- Use `(( ))` for arithmetic
- Check command existence: `command -v git &>/dev/null`

## Functions

- Use `local` for all variables
- Keep functions short and single-purpose
- Return status with `return`, output with `echo`

## Color Codes

Use ANSI color codes; auto-disabled when stdout isn't a TTY (e.g. piped to a logfile, run in CI)

```bash
BOLD=$'\e[1m'; BLUE=$'\e[34m'; GREEN=$'\e[32m'; RED=$'\e[31m'; DIM=$'\e[2m'; RESET=$'\e[0m'
[[ -t 1 ]] || { BOLD=; BLUE=; GREEN=; RED=; DIM=; RESET=; }
```

## Error Handling & Cleanup

```bash
info() { printf '%s●%s %s%s%s\n' "$BLUE" "$RESET" "$BOLD" "$*" "$RESET"; }
ok()   { printf '%s✓%s %s\n' "$GREEN" "$RESET" "$*"; }
err()  { printf '%s✗%s %s\n' "$RED" "$RESET" "$*" >&2; }

# `trap - ERR` disarms the ERR trap below so the explicit `exit 1` doesn't double-report.
die()  { err "$*"; trap - ERR; exit 1; }
trap 'err "script failed at line $LINENO"' ERR
```

## Argument Parsing Pattern

```bash
while [[ $# -gt 0 ]]; do
    case "$1" in
        -h|--help) usage; exit 0 ;;
        -o|--output) output="$2"; shift 2 ;;
        --) shift; break ;;
        -*) die "unknown option: $1" ;;
        *) break ;;
    esac
done
```

## Safe Iteration

```bash
# Over lines (handles whitespace)
while IFS= read -r line; do ...; done < "$file"

# Over globs (handles missing matches)
for f in *.txt; do [[ -e "$f" ]] || continue; ...; done
```

</bash_coding_standards>

<validation>

Always run `shellcheck` cli for static code analysis and lint tool

Usage

```
shellcheck -s bash script.sh
```

Accept disabled shellcheck rules

Add at top of file to disable rules in a file:

```
#!/usr/bin/env bash
# shellcheck disable=SC2003,SC2219
```

add at a specific line to disable line:

```

hexToAscii() {
  # shellcheck disable=SC2059
  printf "\x$1"
}
```

</validation>
