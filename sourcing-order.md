# NetServa 3.0 Shell Sourcing Order

## Standard Sourcing Flow (All Machines)

When a user logs in or starts a new shell, this is the order:

```bash
~/.bashrc (or ~/.zshrc)
  └─> source ~/.rc/_shrc          # 1. Foundation shell functions
        └─> source ~/.rc/_myrc     # 2. User customizations (at end of _shrc)
              └─> source ~/.ns/_nsrc  # 3. NetServa config (optional, at end of _shrc)
```

## Sourcing Order Rationale

1. **`~/.rc/_shrc`** - Foundation layer
   - Provides core aliases, functions, and utilities
   - Cross-platform compatibility
   - Base environment variables
   - Must load FIRST so later files can use its functions

2. **`~/.rc/_myrc`** - User customization layer
   - Local machine-specific settings
   - Can override _shrc defaults
   - Never synced (gitignored)
   - Loads SECOND to allow overrides

3. **`~/.ns/_nsrc`** - NetServa layer (optional)
   - Only exists on central workstation
   - Uses functions from _shrc
   - Adds NetServa-specific environment
   - Loads LAST (most specific)

## The `es` Alias

```bash
alias es='e ~/.rc/_myrc && source ~/.rc/_shrc && source ~/.rc/_myrc && [[ -f ~/.ns/_nsrc ]] && source ~/.ns/_nsrc && echo "✅ Reloaded: _shrc → _myrc → _nsrc"'
```

**What it does:**
1. Edit `~/.rc/_myrc`
2. Reload `~/.rc/_shrc` (foundation)
3. Reload `~/.rc/_myrc` (your edits)
4. Reload `~/.ns/_nsrc` (if exists)
5. Confirm reload

## User Shell RC File Setup

Add to `~/.bashrc` or `~/.zshrc`:

```bash
# Load Runtime Configuration (RC) system
[[ -f ~/.rc/_shrc ]] && source ~/.rc/_shrc

# _shrc automatically loads:
#   - ~/.rc/_myrc (if exists)
#   - ~/.ns/_nsrc (if exists)
```

## Machine Types

### Type 1: Remote Server (Minimal)
```
~/.rc/
├── _shrc              # Synced from workstation
└── _myrc              # Created locally by "rcm init"

Sourcing: _shrc → _myrc
```

### Type 2: Desktop/Laptop (Optional NetServa)
```
~/.rc/
├── _shrc              # Synced or cloned
└── _myrc              # Local customizations

Sourcing: _shrc → _myrc
```

### Type 3: Central Workstation (Full Stack)
```
~/.rc/
├── _shrc              # Foundation
└── _myrc              # Local customizations

~/.ns/
└── _nsrc           # NetServa-specific

Sourcing: _shrc → _myrc → _nsrc
```

## Key Points

- ✅ `_shrc` always loads first (foundation)
- ✅ `_myrc` can override `_shrc` (customization)
- ✅ `_nsrc` loads last (most specific)
- ✅ `_nsrc` is optional (only on workstations with `~/.ns/`)
- ✅ Remote servers never get `_nsrc` (NetServa manages remotely)
