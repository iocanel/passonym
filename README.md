# passonym

Anonymize password store references in dotfiles and configurations.

## Problem

Password store paths often reveal sensitive information:

```bash
# Your email is exposed in public dotfiles
password = $(pass github.com/john.doe@gmail.com/token)
```

## Solution

Create aliases for sensitive paths and use them instead:

```bash
# Anonymized - no personal info exposed
password = $(pass github/token)
```

## Installation

Requires `jq` for JSON handling.

```bash
# The script is already at ~/bin/passonym
# Ensure ~/bin is in your PATH
```

## Usage

### Managing Mappings

Mappings can be **full paths** or **partial paths**:

```bash
# Full path mapping
passonym map add "github.com/john.doe@gmail.com/token" "github/token"

# Partial mapping (just the sensitive directory)
passonym map add "john.doe@gmail.com" "my-email"

# List mappings
passonym map list

# Remove a mapping
passonym map remove "github/token"
```

### Directory Boundary Matching

Partial mappings match on **directory boundaries only**. A directory is either matched whole or not at all.

With mapping `john.doe@gmail.com` → `my-email`:

| Path | Result |
|------|--------|
| `gmail.com/john.doe@gmail.com/password` | `gmail.com/my-email/password` ✓ |
| `github.com/john.doe@gmail.com/token` | `github.com/my-email/token` ✓ |
| `john.doe@gmail.com/password` | `my-email/password` ✓ |

With mapping `john.doe` → `my-username`:

| Path | Result |
|------|--------|
| `gmail.com/john.doe@gmail.com/password` | unchanged ✗ (partial directory) |
| `github.com/john.doe/token` | `github.com/my-username/token` ✓ |

### Creating Symlinks

Creates symbolic links in your password store so aliases actually work with `pass`:

```bash
# Preview what would be created
passonym link --dry-run

# Create the symlinks
passonym link
```

### Removing Symlinks

Remove symlinks that match your mappings:

```bash
# Preview what would be removed
passonym unlink --dry-run

# Remove all symlinks matching mappings
passonym unlink
```

The `unlink` command scans the password store for symlinks containing alias paths and removes them. This is stateless and idempotent.

### Undoing Link Operations

Revert previous link operations using the state file:

```bash
# Undo all previous link operations
passonym undo

# Undo only the last 5 links created
passonym undo 5

# Preview what would be undone
passonym undo --dry-run
```

### Anonymizing Files

Replace real paths with aliases in configuration files:

```bash
# Preview changes (shows diff)
passonym scrub ~/.config/mbsyncrc --dry-run

# Apply changes
passonym scrub ~/.config/mbsyncrc

# Process entire directory
passonym scrub ~/.dotfiles/ --dry-run
```

## Configuration

Mappings are stored in `~/.config/passonym/mappings.json`:

```json
{
  "github.com/john.doe@gmail.com/token": "github/token",
  "john.doe@gmail.com": "my-email"
}
```

## Supported Patterns

The `scrub` command handles these pass invocation styles:

- `pass path/to/secret`
- `pass "path/to/secret"`
- `pass 'path/to/secret'`

## Testing

Run the test suite:

```bash
passonym --test
```
