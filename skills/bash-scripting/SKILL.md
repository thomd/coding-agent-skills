---
name: bash-scripting
description: Guidelines for writing production-quality bash scripts. Use whenever asked to write a bash script.
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

- Write output to stdout by default; support `--output <file>`
- Write errors and diagnostics to stderr only
- Accept stdin when `-` is given as filename
- Produce one record per line for text output (pipeline-friendly)
- Offer `--json` for machine-parseable output when appropriate

## Arguments & Flags

- Required values → positional arguments
- Optional values → flags with sensible defaults
- Support `--` to separate flags from positional arguments
- Provide short flags for common operations (`-v`, `-h`, `-o`)

## Exit Codes

- `0` = success
- `1` = general error
- `2` = invalid usage

## Error Messages

- Format: `scriptname: error: what went wrong`
- Include what happened and how to fix it
- Suggest `--help` on invalid usage

## Robustness

- Validate arguments before doing any work
- Fail fast—don't partially complete then error
- Clean up temp files on exit (use trap)

</cli_design_principles>

Be idempotent where possible (safe to run twice)

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

## Error Handling & Cleanup

```bash
cleanup() { rm -f "$tmpfile"; }
trap cleanup EXIT

die() { echo "${0##*/}: error: $*" >&2; exit 1; }
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

<template>
Use this structure as a starting point:

```bash
#!/usr/bin/env bash
set -euo pipefail

readonly SCRIPT_NAME="${0##*/}"
readonly VERSION="1.0.0"

usage() {
    cat >&2 <<EOF
Usage: $SCRIPT_NAME [OPTIONS] <required_arg>

Brief description of what this script does.

Arguments:
    required_arg    Description of required argument

Options:
    -h, --help      Show this help message
    -v, --verbose   Enable verbose output
    -o, --output    Output file (default: stdout)

Examples:
    $SCRIPT_NAME input.txt
    $SCRIPT_NAME -o result.txt input.txt
    echo "data" | $SCRIPT_NAME -
EOF
}

die() {
    echo "$SCRIPT_NAME: error: $*" >&2
    exit 1
}

main() {
    local verbose=false
    local output="/dev/stdout"

    while [[ $# -gt 0 ]]; do
        case "$1" in
            -h|--help) usage; exit 0 ;;
            -v|--verbose) verbose=true; shift ;;
            -o|--output) output="$2"; shift 2 ;;
            --) shift; break ;;
            -*) die "unknown option: $1 (see --help)" ;;
            *) break ;;
        esac
    done

    [[ $# -ge 1 ]] || { usage; exit 2; }

    local input="$1"

    # Check dependencies
    # command -v jq &>/dev/null || die "jq is required but not installed"

    # Main logic here
}

main "$@"
```

</template>

<output_format>
Provide:

1. The complete bash script
2. 2-3 sentences explaining key design decisions
3. Example commands showing typical usage
   </output_format>

<validation>
Remind the user to validate with:
```bash
shellcheck -s bash script.sh
```
</validation>
