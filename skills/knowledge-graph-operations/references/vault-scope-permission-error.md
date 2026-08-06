# Vault scope permission error

## Failure signature

A `kg chat` invocation launched without an explicit vault root may use the process working directory as `vault_root`. When the working directory is the user's home directory, recursive Markdown discovery can encounter protected macOS bundles, for example:

```text
Dir.glob(... "**/*.md")
Operation not permitted - /Users/user/Pictures/Photos Library.photoslibrary
```

The important clue is that the denied path is outside the intended knowledge vault. This is a scope-selection error, not evidence that Photos access is required.

## Root cause trace

- CLI initialization defaults `vault_root` to `Dir.pwd`.
- The repository recursively discovers `**/*.md` under that root.
- Therefore launching from `$HOME` broadens discovery to the whole home directory.

## Confirmed correction

Use the global `--vault` option before the subcommand:

```bash
kg --vault /Users/user/vaults/KnowledgeGraph chat \
  --text 'message' \
  --source telegram \
  --source-type chat \
  --conversation '<id>' \
  --sender '<id>' \
  --json
```

A corrected run returned structured JSON with `status: ok` and `route: observe`, without touching the Photos library.

## Do not do this

- Do not grant Photos or Full Disk Access merely to silence the error.
- Do not add a broad exclusion for one protected directory while retaining `$HOME` as the vault; another unrelated directory can fail next.
- Do not place `--vault` after `chat`; it is a global option.
- Do not run `kg chat --json` without exactly one of `--text`, `--stdin`, or `--file`.
