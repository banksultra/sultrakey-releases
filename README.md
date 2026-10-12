# sultrakey releases

Encrypted `.env` files (age X25519) and a launcher that runs an application with the decrypted values.

This repository holds release files only. The source code is private.

## Install

Windows (PowerShell, no Administrator rights needed):

```powershell
powershell -ExecutionPolicy Bypass -c "irm https://github.com/banksultra/sultrakey-releases/releases/latest/download/install.ps1 | iex"
```

Linux / macOS:

```sh
curl -fsSL https://github.com/banksultra/sultrakey-releases/releases/latest/download/install.sh | sh        # laptop
curl -fsSL https://github.com/banksultra/sultrakey-releases/releases/latest/download/install.sh | sudo sh   # server → /usr/bin
```

A given version: `... | SULTRAKEY_VERSION=0.6.1 sh` (Windows: `$env:SULTRAKEY_VERSION = '0.6.1'` first).
Releases start at 0.6.1.

## Update

```sh
sultrakey update --check     # only checks; exit 1 when a newer version exists
sultrakey update             # servers: sudo sultrakey update
```

Installed 0.6.0 or older? Run the install command above once; keys and `.env` files are not touched.

## Install by hand (servers without internet)

1. From the [latest release](https://github.com/banksultra/sultrakey-releases/releases/latest), download
   the binary for the server (`sultrakey-x86_64-unknown-linux-musl`, ARM:
   `sultrakey-aarch64-unknown-linux-musl`) and `SHA256SUMS`.
2. Copy both to the server, then check the binary. It must print `OK`:

   ```sh
   grep 'sultrakey-x86_64-unknown-linux-musl$' SHA256SUMS | sha256sum -c -
   ```

3. Install:

   ```sh
   chmod +x sultrakey-x86_64-unknown-linux-musl
   sudo ./sultrakey-x86_64-unknown-linux-musl install
   ```

## Files in each release

| File                        | Use                        |
| --------------------------- | -------------------------- |
| `sultrakey-<target>[.exe]`  | the binary, one per system |
| `SHA256SUMS`                | checksums of every file    |
| `manifest.json`             | read by `sultrakey update` |
| `install.sh`, `install.ps1` | the install scripts above  |

## License

[MIT](LICENSE) © 2026 Rede
