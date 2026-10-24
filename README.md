# goplater-action

```yaml
# Goplater version to use (default: 'latest')
version: latest

# Recursively process files
recursive: false

# Flatten output paths without preserving source directory structure
flatten: false

# Source root; output keeps paths relative to it (default is inferred)
root: ""

# File or directory to template (required)
path: ""

# Source directory containing files
source: "."

# Output directory for templated files
output: "."

# Allowed output directories, including their subdirectories (comma-separated list)
allowed-outputs: ""

# Regular expressions used to match files for processing
match: ".*"

# Experimental: maximum blank lines kept around non-output actions (-1 disables trimming)
trim-max-blank-lines: -1

# Show what would be written without changing anything
dry-run: false

# Continue processing when errors occur
ignore-errors: false
```
