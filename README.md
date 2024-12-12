# Super Mario 64 Wii/Gamecube Port

This repo does **not** include all assets necessary for compiling the game.
A prior copy of the game is required to extract the assets.

## Known Issues

This is a work-in-progress, most things are broken. It *does* run on a real hardware.

Using OpenGX throws an invalid read on Dolphin, though ignoring that warning things render to some extend. Use `ENABLE_OPENGX=1` to enable it

**Broken (Native GX):**
 - The Z buffer
 - Only implemented basic color/texture shaders

**Broken (OpenGX):**

OpenGX is still a work in progress, though it renders a bit better than Native GX.
 - When a dialog is opened, the top half of screen flickers
 - Vertices are notoriously broken specially when the camera is up close
 - Mario's face ingame and other decals don't render properly

**Somewhat Working:**
 - Audio

**Working:**
 - Controls
 - Saving

## Building

Successful compilation will result in a `boot.dol` being created in `build/VERSION_GX/boot.dol` where `VERSION` is one of `us`, `eu`, `jp` or `sh`, and `GX` is either `wii` or `cube`.

Place the `boot.dol` and `meta.xml` from `build/us/wii` in a directory called `/apps/sm64` on your SDCARD and run using the [Homebrew Channel](https://wiibrew.org/wiki/Homebrew_Channel).

**Supported Build Methods:**

  - [Docker](#docker)
  - [Linux / WSL (Ubuntu 18.04 or higher)](#linux--wsl-ubuntu)
  - [Windows (MSYS2)](#windows-msys2)

### Docker

The following assumes a basic understanding of [Docker](https://www.docker.com/); if you do not belong to the `docker` group, prefix those commands with `sudo`.

**Clone Repository:**

```sh
git clone https://github.com/mkst/sm64-port.git --branch wii
```

**Navigate into freshly checked out repo:**

```sh
cd sm64-port
```

**Copy in baserom.XX.z64:**

```sh
cp /path/to/your/baserom.us.z64 ./ # change 'us' to 'eu', 'jp' or 'sh' as appropriate
```

**Build with pre-baked image:**

Change `VERSION=us` if applicable. If on Windows replace `$(pwd):/sm64` with the path to the current directory, e.g. `"C:\path\to\sm64-port:/sm64"`.
```sh
docker run --rm -v $(pwd):/sm64 markstreet/sm64:wii make VERSION=us --jobs 4 # Linux/OSX
```

### Linux / WSL (Ubuntu)

Tested successfully on **Ubuntu 18.04** and **20.04**. Does not work on **16.04**. (Note, these are outdated, refer to devkitpro's page to install)

https://devkitpro.org/wiki/devkitPro_pacman


```sh
sudo su -

apt-get update && \
    apt-get install -y \
        binutils-mips-linux-gnu \
        bsdmainutils \
        build-essential \
        libaudiofile-dev \
        pkg-config \
        python3 \
        wget \
        zlib1g-dev

wget https://github.com/devkitPro/pacman/releases/download/v1.0.2/devkitpro-pacman.amd64.deb \
  -O devkitpro.deb && \
  echo ebc9f199da9a685e5264c87578efe29309d5d90f44f99f3dad9dcd96323fece3 devkitpro.deb | sha256sum --check && \
  apt install -y ./devkitpro.deb && \
  rm devkitpro.deb

dkp-pacman -Syu wii-dev --noconfirm
# if this ^^ fails with error about archive format, use a VPN to get yourself out of the USA and then try again.

# For OpenGX
dkp-pacman -Syu gamecube-sdl2 gamecube-opengx wii-sdl2 wii-opengx --noconfirm

exit

cd

git clone https://github.com/mkst/sm64-port.git --branch wii

cd sm64-port

# go and copy the baserom to c:\temp (create that directory in Windows Explorer)
cp /mnt/c/temp/baserom.us.z64 ./

sudo chmod 644 ./baserom.us.z64

export PATH="/opt/devkitpro/tools/bin/:~/sm64-port/tools:${PATH}"
export DEVKITPRO=/opt/devkitpro
export DEVKITPPC=/opt/devkitpro/devkitPPC

# Note, use ENABLE_OPENGX=1 to use OpenGL wrapper
make -j4 TARGET_WII=1
```

### Windows (MSYS2)

WSL is the preferred route, but you can also use MSYS2 (MINGW64) to compile.

For each instruction copy and paste the contents into the **MING64** console.

**Get MSYS2:**

Navigate to https://www.msys2.org/ and download the installer.

**Install and Run MINGW64:**

```
Next, Next, Next, Finish (keep the box checked to "Run MSYS 64bit now").
```

**Add keyserver for package validation:**

```sh
pacman-key --recv BC26F752D25B92CE272E0F44F7FD5492264BB9D0 --keyserver keyserver.ubuntu.com
pacman-key --lsign BC26F752D25B92CE272E0F44F7FD5492264BB9D0
```

**Add DevKitPro keyring:**

```sh
pacman -U --noconfirm https://downloads.devkitpro.org/devkitpro-keyring.pkg.tar.xz
```

**Add DevKitPro package repositories:**

```sh
cat <<EOF >> /etc/pacman.conf
[dkp-libs]
Server = https://downloads.devkitpro.org/packages
[dkp-windows]
Server = https://downloads.devkitpro.org/packages/windows
EOF
```

**Update dependencies:**

```sh
pacman -Syu --noconfirm
```

MINGW64 may close itself when done, if it does, find `MSYS2 MinGW 64bit` in your Start Menu and open again.

**Install Dependencies:**

```sh
pacman -S wii-dev git make python3 mingw-w64-x86_64-gcc --noconfirm
```

Note, for OpenGX you have to add these additional dependencies:
```sh
pacman -S gamecube-sdl2 gamecube-opengx wii-sdl2 wii-opengx --noconfirm
```

**Setup Environment Variables:**

```sh
export PATH=$PATH:/opt/devkitpro/tools/bin && echo "OK!"
export DEVKITPRO=/opt/devkitpro && echo "OK!"
export DEVKITPPC=/opt/devkitpro/devkitPPC && echo "OK!"
```

**Clone Repository:**

```sh
git clone https://github.com/mkst/sm64-port.git --branch wii
```

**Navigate into freshly checked out repo:**

```sh
cd sm64-port && echo "OK!"
```

**Copy in baserom.XX.z64:**

This assumes that you have create the directory `c:\temp` via Windows Explorer and copied the Super Mario 64 `baserom.XX.z64` to it.
```sh
cp /c/temp/baserom.us.z64 ./ && echo "OK!" # change 'us' to 'eu', 'jp' or 'sh' as appropriate
```

**Compile:**

```sh
make VERSION=us TARGET_WII=1 --jobs 4 # Note, use ENABLE_OPENGX=1 to use OpenGL wrapper
```

### Other Operating Systems

TBD; feel free to submit a PR.

## Project Structure

    sm64
    ├── actors: object behaviors, geo layout, and display lists
    ├── asm: handwritten assembly code, rom header
    │   └── non_matchings: asm for non-matching sections
    ├── assets: animation and demo data
    │   ├── anims: animation data
    │   └── demos: demo data
    ├── bin: C files for ordering display lists and textures
    ├── build: output directory
    ├── data: behavior scripts, misc. data
    ├── doxygen: documentation infrastructure
    ├── enhancements: example source modifications
    ├── include: header files
    ├── levels: level scripts, geo layout, and display lists
    ├── lib: SDK library code
    ├── rsp: audio and Fast3D RSP assembly code
    ├── sound: sequences, sound samples, and sound banks
    ├── src: C source code for game
    │   ├── audio: audio code
    │   ├── buffers: stacks, heaps, and task buffers
    │   ├── engine: script processing engines and utils
    │   ├── game: behaviors and rest of game source
    │   ├── goddard: Mario intro screen
    │   ├── menu: title screen and file, act, and debug level selection menus
    │   └── pc: port code, audio and video renderer
    ├── text: dialog, level names, act names
    ├── textures: skybox and generic texture data
    └── tools: build tools

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.
