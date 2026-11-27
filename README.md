# goplater-action

Run [Goplater](https://github.com/CodeShellDev/goplater) in your GitHub Actions workflows.

Goplater templates files and directories using Go templates, with support for recursive processing, output path control, matching, and automatic blank-line handling.

> [!IMPORTANT]
> Searching for **Goplater Bot**?
> See [`goplater-bot`](https://github.com/CodeShellDev/goplater-action/BOT.md)!

## Usage

The simplest setup is:

```yaml
name: Generate files

on:
  push:

jobs:
  generate:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: CodeShellDev/goplater-action@v1
```

By default, this templates the current directory and writes the generated files back to the current directory.

### Template a directory

```yaml
- uses: CodeShellDev/goplater-action@v1
  with:
    path: templates/
    output: generated/
    recursive: true
```

### Template a single file

```yaml
- uses: CodeShellDev/goplater-action@v1
  with:
    path: templates/config.yaml
    output: generated/
```

## Inputs

| Input                  | Description                                                        | Default  |
| ---------------------- | ------------------------------------------------------------------ | -------- |
| `path`                 | File or directory to template                                      | `.`      |
| `version`              | Goplater version to use (`latest-prerelease` includes prereleases) | `latest` |
| `source`               | Source directory containing files                                  | `.`      |
| `output`               | Output directory for templated files                               | `.`      |
| `recursive`            | Recursively process files                                          | `false`  |
| `flatten`              | Flatten output paths                                               | `false`  |
| `root`                 | Source root; output paths are relative to it (default is inferred) | `""`     |
| `match`                | Regexes used to match files (one per line)                         | `.*`     |
| `exclude`              | Regexes used to exclude files (one per line)                       | `""`     |
| `allowed-inputs`       | Comma-separated allowed input directories                          | `""`     |
| `allowed-outputs`      | Comma-separated allowed output directories                         | `""`     |
| `trim-max-blank-lines` | Maximum blank lines kept around non-output actions (`-1` = off)    | `-1`     |
| `dry-run`              | Show what would be written without changing anything               | `false`  |
| `ignore-errors`        | Continue processing when errors occur                              | `false`  |

### Version Selection

The `version` input accepts:

| Value               | Description                    |
| ------------------- | ------------------------------ |
| `latest`            | Latest stable Goplater release |
| `latest-prerelease` | Latest (includes prereleases)  |
| `v1.2.3`            | A specific release tag         |

The action resolves version labels such as `latest` and `latest-prerelease` to the actual GitHub release tag before downloading the binary.

The resolved tag is exposed through the `version` output.

For example, if `version` is set to `latest` and the latest stable release is `v1.6.2`, the output will be `v1.6.2`, not `latest`.

## Outputs

The action exposes information about the Goplater binary that was downloaded.

| Output         | Description                                   | Example                       |
| -------------- | --------------------------------------------- | ----------------------------- |
| `version`      | Actual Goplater release tag used              | `v1.6.2`                      |
| `os`           | Operating system target used for the download | `linux`                       |
| `arch`         | CPU architecture target used for the download | `amd64`                       |
| `asset`        | Name of the downloaded release asset          | `goplater-v1.6.2-linux-amd64` |
| `download-url` | URL used to download the release asset        | GitHub release asset URL      |

## Examples

### Recursive templates

Given:

```text
templates/
├── config/
│   └── app.yaml
└── README.md
```

Run:

```yaml
- uses: CodeShellDev/goplater-action@v1
  with:
    path: templates/
    output: generated/
    recursive: true
```

Goplater processes the files recursively and preserves their relative structure in the output.

### Flatten output

Use `flatten` when you don't want the source directory structure preserved:

```yaml
- uses: CodeShellDev/goplater-action@v1
  with:
    path: templates/
    output: generated/
    recursive: true
    flatten: true
```

### Match specific files

`match` accepts regular expressions used to determine which files are processed.

For example:

```yaml
- uses: CodeShellDev/goplater-action@v1
  with:
    path: templates/
    output: generated/
    recursive: true
    match: |
      .*\\.yaml$
```

This processes only files ending in `.yaml`.

### Restrict output directories

Use `allowed-outputs` to restrict where Goplater is allowed to write:

```yaml
- uses: CodeShellDev/goplater-action@v1
  with:
    path: templates/
    output: generated/
    allowed-outputs: "generated,dist"
```

Multiple directories are comma-separated:

```yaml
allowed-outputs: "generated,dist,public"
```

## Blank-line handling

Goplater can automatically remove unnecessary blank lines around template actions that don't produce output.

For example, instead of having to write:

```gotemplate
{{- $value := .Value -}}
```

you can enable automatic blank-line handling:

```yaml
- uses: CodeShellDev/goplater-action@v1
  with:
    path: templates/
    trim-max-blank-lines: 1
```

`trim-max-blank-lines` controls how many blank lines are allowed to remain at the junction.

Use `0` to remove all blank lines:

```yaml
trim-max-blank-lines: 0
```

The default is `-1`, which disables the feature.

## Version

By default, the action uses the latest Goplater release:

```yaml
- uses: CodeShellDev/goplater-action@v1
```

You can pin a specific Goplater version:

```yaml
- uses: CodeShellDev/goplater-action@v1
  with:
    version: v1.2.3
```

Pinning a version is recommended for workflows where reproducible output is important.

## Source and output

`path` determines **what Goplater processes**.

`source` determines the source directory used when resolving files.

`output` determines **where generated files are written**.

For example:

```yaml
- uses: CodeShellDev/goplater-action@v1
  with:
    path: templates/
    source: .
    output: generated/
    recursive: true
```

This processes `templates/` and writes the generated files under `generated/`.

## Requirements

The action is intended to run after checking out your repository:

```yaml
steps:
  - uses: actions/checkout@v4

  - uses: CodeShellDev/goplater-action@v1
    with:
      path: templates/
      output: generated/
      recursive: true
```

The action downloads the requested Goplater release automatically, so Goplater does not need to be installed separately.
