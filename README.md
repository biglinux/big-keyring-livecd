# big-keyring-livecd

The pacman keyring of the BigLinux live session, ready to use.

> [!WARNING]
> Only for the ISO build. Do not install it on a regular system: it overwrites
> `/etc/pacman.d/gnupg/pubring.gpg` and `trustdb.gpg` with its own copy.

It takes the place of `biglinux-livecd-key` (same files, new name): the
package provides, conflicts with and replaces it, so an upgrade swaps one for
the other.

## Why

A live session starts with an empty pacman keyring, and building one with
`pacman-key --init` and `--populate` on every boot is slow and needs every
keyring package installed. This package ships the keyring already built and
trusted, so the live session can verify packages (Calamares installing the
system, or `pacman -S` in the live session) as soon as it boots.

## What it installs

| Path | What |
|---|---|
| `/usr/share/pacman/keyrings/pubring.gpg` | the public keys (Arch Linux, Manjaro, BigLinux) |
| `/usr/share/pacman/keyrings/trustdb.gpg` | their trust |
| `/usr/bin/big-keyring-livecd.sh` | copies both into `/etc/pacman.d/gnupg/` and runs `pacman-key --init` |
| `/usr/lib/systemd/system/big-keyring-livecd.service` | runs that script at boot, after `pacman-init.service` |

Installing or upgrading the package runs the same script once.

## How the keyring is built

The [Update Key](.github/workflows/update-key.yml) workflow runs every
Wednesday at 23:00 UTC, on every push, and by hand (*Run workflow*). In a
`manjarolinux/base` container it:

1. installs `archlinux-keyring` and `manjaro-keyring` from Manjaro's
   repositories, and the keys of
   [biglinux-keyring](https://github.com/biglinux/biglinux-keyring);
2. runs `pacman-key --init` and `pacman-key --populate archlinux manjaro biglinux`,
   and checks that the BigLinux signing key
   `F7EB9C2849EFFEFF05789738047C6A782618D2CA` is there;
3. commits `pubring.gpg` and `trustdb.gpg` when they changed;
4. asks [biglinux/build-package](https://github.com/biglinux/build-package)
   to build and publish the package in the `stable` repository of
   [packages.biglinux.com.br](https://packages.biglinux.com.br). The weekly run
   builds only when the keyring changed.

A key added to one of those keyrings reaches the next ISO without any change
here: the next run picks it up.

The build request needs the secret `BUILD_PACKAGE_TOKEN` (best set once for
the whole biglinux organization): a fine-grained GitHub token with access to
`biglinux/build-package` only and the permission *Contents: Read and write*.

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
