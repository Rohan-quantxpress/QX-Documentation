# Command-Line Usage

## Syntax

```bash
python generate_report.py [OPTIONS]
```

## Arguments

| Argument | Default | Description |
|---|---|---|
| `--config` | `process_config.json` | Process configuration file |
| `--out` | `report.md` | Markdown output path |
| `--html-out` | Derived from `--out` | HTML output path |
| `--json-out` | None | Optional raw JSON output |

## Default Execution

```bash
python generate_report.py
```

Default outputs:

```text
report.md
report.html
```

## Custom Configuration

```bash
python generate_report.py \
    --config process_config.json \
    --out report.md
```

## Custom HTML Output

```bash
python generate_report.py \
    --config process_config.json \
    --out report.md \
    --html-out health_report.html
```

## Generate JSON Status

```bash
python generate_report.py \
    --config process_config.json \
    --out report.md \
    --json-out status.json
```

## Output Files

Depending on the supplied arguments, the tool can generate:

```text
report.md
report.html
status.json
```