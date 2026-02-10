# passonym

Anonymize password store references in dotfiles and configurations.

## Problem

Password store paths often reveal sensitive information:

```bash
# Your email is exposed in public dotfiles
password = $(pass Email/john.doe@acme-corp.com)
```

## Solution

Create aliases for sensitive paths and use them instead:

```bash
# Anonymized - no personal info exposed
password = $(pass work-email)
```

## Installation

Requires `jq` for JSON handling.

```bash
# The script is already at ~/bin/passonym
# Ensure ~/bin is in your PATH
```

## Usage

### Managing Mappings

```bash
# Add a mapping
passonym map add "Email/john.doe@acme.com" "work-email"
passonym map add "Services/github/johndoe" "github"

# List mappings
passonym map list

# Remove a mapping
passonym map remove "work-email"
```

### Creating Symlinks

Creates symbolic links in your password store so aliases actually work with `pass`:

```bash
# Preview what would be created
passonym link --dry-run

# Create the symlinks
passonym link
```

### Anonymizing Files

Replace real paths with aliases in configuration files:

```bash
# Preview changes (shows diff)
passonym scrub ~/.config/mbsyncrc --dry-run

# Apply changes
passonym scrub ~/.config/mbsyncrc

# Process entire directory
passonym scrub ~/.config/email/ --dry-run
```

## Configuration

Mappings are stored in `~/.config/passonym/mappings.json`:

```json
{
  "Email/john.doe@acme.com": "work-email",
  "Services/github/johndoe": "github"
}
```

## Supported Patterns

The `scrub` command handles these pass invocation styles:

- `pass Path/to/secret`
- `pass "Path/to/secret"`
- `pass 'Path/to/secret'`

## Testing

Run the test suite:

```bash
passonym --test
```
