# O·LESA OS updates

System updates for O·LESA OS are published here as GitHub Releases. This repository holds no source code: only signed release assets, the public half of the signing key and this description.

An installed O·LESA OS checks

```
https://github.com/o-lesa/olesa-os-updates/releases/latest/download/manifest.txt
```

and downloads the files that manifest names from the same release. Settings → Update and the console command `update` do this. The address can be changed with `update url <base-url/>`.

## Release assets

Every release carries:

- `manifest.txt`, the signed description of the release;
- the files it lists: `kernel.efi` always, and optionally other system files such as `player.olesa` or `browser.olesa` for `system/apps/`.

```
OLESA-UPDATE 1
version 0.9.0
notes One-line description
file system/kernel.efi 24766464 <sha256 hex> kernel.efi
file system/apps/player.olesa 3113336 <sha256 hex> player.olesa
sig <128 hex: ECDSA P-256 r || s over SHA-256 of every line above>
```

The OS checks the signature with the public key compiled into its kernel before it reads anything else in the manifest. It accepts only a version newer than the one it runs, checks every file's size and SHA-256, and tries the new kernel once before making it the default. A release that fails any of these checks is never installed.

## Signing key

- `update-signing-key.pem`: the ECDSA P-256 public key, in PEM form.
- `update-signing-key.hex`: the same key as an uncompressed point, as compiled into the kernel.

The private key never leaves the maintainer's machine and is not in this or any other repository. Assets uploaded here without a valid signature are refused by every installed system.

## Publishing a release

From the O·LESA OS source tree:

```sh
python3 scripts/build.py --openzfs-core
python3 scripts/update_sign.py package --kernel build/openzfs-core/BOOTX64.EFI --version 0.9.0 \
    --file system/apps/player.olesa=build/olesa/player.olesa \
    --file system/apps/browser.olesa=build/olesa/browser.olesa \
    --notes "..." --out dist/update-0.9.0
python3 scripts/update_sign.py verify dist/update-0.9.0/manifest.txt
gh release create v0.9.0 --repo o-lesa/olesa-os-updates --title "O·LESA OS 0.9.0" \
    dist/update-0.9.0/manifest.txt dist/update-0.9.0/kernel.efi \
    dist/update-0.9.0/player.olesa dist/update-0.9.0/browser.olesa
```

The release version must be higher than `OS_VERSION` of the systems that should take it. The newest non-prerelease release is the one `latest/download` serves.
