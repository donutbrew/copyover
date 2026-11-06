# copyover - `co` script

A shell script wrapper around `cp` that automatically detects Samba/SMB mounts and uses `rsync over ssh` for more reliable remote transfers.

## Usage

```bash
co [cp options] source dest
```

## How It Works

1. **Detects Samba mounts** using `df -t cifs,smbfs` to check if source or destination are on network mounts
2. **Extracts hostname** from the mount source (format: `//hostname/share`)
3. **Falls back to `cp`** if both paths are local
4. **Uses `rsync -avz --delete`** when one path is remote and the other is local
5. **Assumes path structure is the same** locally and remotely (e.g., `/Users/fep2/files` exists on both systems)

## Examples

### Local copy (uses regular `cp`)
```bash
co file.txt /tmp/file.txt
```

### Remote to local (uses `rsync over ssh`)
```bash
co /mnt/smb/remote/file.txt ./file.txt
```
Automatically converts to: `rsync -avz --delete remote-host:/mnt/smb/remote/file.txt ./file.txt`

### Local to remote (uses `rsync over ssh`)
```bash
co ./large_folder /mnt/smb/remote/destination
```
Automatically converts to: `rsync -avz --delete ./large_folder remote-host:/mnt/smb/remote/destination`

### With cp options
```bash
co -r dir_source /mnt/smb/remote/dir_dest
```

## Requirements

- `bash` 4+
- `df` (standard on Unix/Linux/macOS)
- `rsync` (for remote transfers)
- SSH access to the remote host (for rsync over ssh)
- Mounted Samba/SMB shares (detected via `df -t cifs,smbfs`)

## Notes

- Only handles **local-to-remote** or **remote-to-local** transfers via rsync
- If both source and dest are remote, falls back to regular `cp`
- cp options are passed through to either `cp` or `rsync` as-is
- The `--delete` flag is used by default with rsync to match cp behavior
