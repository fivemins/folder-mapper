# folder-mapper

Temporary folder mapping tool with security features for AI agents.

## Features

- 🔗 **Mount external folders** into workspace via symlinks
- 🔒 **Read-only mode** by default
- 🛡️ **System directory protection** - blocks dangerous paths
- ⚙️ **User-configurable** forbidden and sensitive paths
- ⚠️ **Confirmation prompts** for sensitive operations

## Installation

```bash
npx skills add fivemins/folder-mapper
```

## Usage

### Map a folder

```bash
python3 scripts/map_folder.py mount "/path/to/folder"
```

### List current mappings

```bash
python3 scripts/map_folder.py list
```

### Unmount

```bash
python3 scripts/map_folder.py unmount <folder_name>
```

### Configure forbidden paths

```bash
# Add directory that cannot be mapped
python3 scripts/map_folder.py forbid "/path/to/secure"

# Add directory requiring confirmation for modifications
python3 scripts/map_folder.py sensitive "/path/to/important"

# View configuration
python3 scripts/map_folder.py config
```

## Security

- Default forbidden: `/`, `/bin`, `/etc`, `/proc`, etc.
- User can add custom forbidden/sensitive paths
- Read-only mapping by default
- Confirmation required for delete/modify operations on sensitive directories

## License

MIT
