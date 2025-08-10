# Homebrew as a Flatpak SDK extension

[Homebrew](https://brew.sh) is a user-level package manager for MacOS and Linux systems. We can install many dev-depencencies from brew, such as rust, gcc, golang, and more, and this inspired me to use it for development inside the [Visual Studio Code Flatpak](https://flathub.org/apps/com.visualstudio.code), since for development, having a package manager around is super helpful.

## Building/Installation

Install flatpak-builder from your package manager.

```shell
sudo dnf install flatpak-builder
```

Clone source

```shell
git clone https://github.com/ryanabx/org.freedesktop.Sdk.Extension.brew.git
cd org.freedesktop.Sdk.Extension.brew
```

Run flatpak-builder:

```shell
# User installation
flatpak-builder --user --install --force-clean .build org.freedesktop.Sdk.Extension.Homebrew.json
# System installation
flatpak-builder --system --install --force-clean .build org.freedesktop.Sdk.Extension.Homebrew.json
```

## Usage

Once installed, there are 2 ways to enable the extension in a flatpak.

In a shell from within the flatpak, you can run the enable script manually:

```shell
# Within the target flatpak
/usr/lib/sdk/Homebrew/enable.sh
```

OR, some flatpaks, such as [vscode](https://github.com/flathub/com.visualstudio.code?tab=readme-ov-file#support-for-language-extension), include an environment variable to set at startup which will enable it automatically.

```shell
# On the host
FLATPAK_ENABLE_SDK_EXT=Homebrew flatpak run com.visualstudio.code
```

## How it works

We follow the custom installation procedure outlined at https://docs.brew.sh/Installation#untar-anywhere-unsupported

The Homebrew environment is set up inside `$XDG_DATA_HOME/linuxbrew/.linuxbrew/` using the following procedure:

1. When the SDK is built, homebrew is cloned into the `/usr/lib/sdk/Homebrew/.linuxbrew/` directory.
2. When the SDK is installed, a script to enable homebrew is at `/usr/lib/sdk/Homebrew/enable.sh`

3. The user enables the SDK by executing the `/usr/lib/sdk/Homebrew/enable.sh` script.
4. The script will copy the Homebrew installation to the `$XDG_DATA_HOME/linuxbrew/.linuxbrew/` directory, and enable Homebrew. You still need to run `enable.sh` on each consecutive flatpak run or use the proper environment variable.

## Caveats

From https://docs.brew.sh/Installation#untar-anywhere-unsupported :

> Technically, you can just extract (or git clone) Homebrew wherever you want. However, you shouldn’t install outside the default, supported, best prefix. Many things will need to be built from source outside the default prefix. Building from source is slow, energy-inefficient, buggy and unsupported. The main reason Homebrew just works is because we use bottles (binary packages) and most of these require using the default prefix. If you decide to use another prefix: don’t open any issues, even if you think they are unrelated to your prefix choice. They will be closed without response.

You will find that for most package installs, brew will manually build from source, because the binary caches online expect the path to be `/home/linuxbrew/.linuxbrew/`.

We cannot use this path, because some applications mount the host's `/home` into the flatpak sandbox. We instead install homebrew at `$XDG_DATA_HOME/linuxbrew/.linuxbrew/`.

We still have a fully functional homebrew environment, but we run into that one issue.