# How to get qlscribe to work on modern Arch Linux.

qlscribe is an old lighscribe program, however I think it is the ONLY linux lightscribe label design tool. Due to its old-ness, there will be alot of work to get it working.\

WARNING: I used AI to help me with this because I don't know C++ and I don't want to bother someone with getting an old obsolete piece of software working. Sorry.

## Installing LaCie

I recommend getting LaCie, I installed it first, and got it to work.\
Clone the repository. 

```
# git clone https://aur.archlinux.org/bin32-4l.git
```
LaCie wants to use gksudo, but it's been long abandoned and it is not available. Replace the contents of `4L-cli` in the cloned directory with:
```
#!/usr/bin/bash
exec /opt/4L/4L-cli.bin "$@"
```
Also, remove the gksudo dependency in the PKGBUILD, update the release number and change the MD5 sums:
```
pkgrel=12
depends=('lib32-fontconfig' 'lib32-gcc-libs' 'lib32-libxcursor' 'lib32-libxi' 'lib32-libxinerama' 'lib32-libxrandr' 'bin32-lightscribe')
md5sums=('11fc8b2daeaed2b61a567056413bdefd'
         '582b6cce827c8909115a31c7bc10d6ce'
         '48d1af9e30a21f6ec1f8c85151b3a652'
         '93d44c0a7b29ecac19a9e9d728a16954') # this line needs to be updated, leave the other ones alone.
```
There is a comment on the AUR page of LaCie suggesting a propper fix for this problem, but honestly I'm too dumb to understand that. This makes the app work, however you'll need to run it as root to burn a disc. After that it should be able to build correctly
```
# makepkg -si
```
## Installing QLScribe 0.16

1. Get the correct version, 0.16 from https://sourceforge.net/projects/qlscribe/ and untar it. We'll be doing plenty of work with these sources.
2. Install qt4. Again, I'm too dumb to rewrite this for qt5 so you'll need to install qt4. As of now, there isn't a pre-compiled binary for that.
3. We'll need additional lightscribe library files, that aren't included in the AUR package. Download an RPM file, that has these files, from here: https://archive.org/details/lightscribePublicSDK-1.14.32.1-linux-2.6-intel. Make sure to decompress this file with tar. Also make sure that you download the "lightscribePublicSDK-..." file, not the "lighscribe-..." file as that one contains files we already have.
4. Go into the unpacked RPM file and change directory into /usr/include (which is inside the unpacked RPM file)
5. Copy all three files (`lighscribe.h`, `lighscribe_cxx.h`, `lightscibe_errors.h`) into the QLScribe source directory.
6. In the `src/qlightscribe.h` and the `src/qlightscribe.cpp` files, remove the obsolete `throw( QString )`. DO NOT REMOVE THE WHOLE LINE, remove only the `throw ( Qstring )`s.
7. In lscribed/dbuscpp.cpp add the line, the compiler will refuse to compile without that:
```
#include <ctime>
```
8. Configure the application with:
```
# cmake -S . -B build \
  -DLSCRIBE_INCLUDE_DIR="PATH_TO_SOURCES" \
  -DLSCRIBE_LIBRARY="/usr/lib32/liblightscribe.so"
```
Change PATH_TO_SOURCES, to the path where the sources reside. E. g. $HOME/qlscribe-0.16

9. Compile the application:

```
cmake --build build -j$(nproc)
```

If everything went correctly, and the program isn't even more outdated when you're trying this it should compile correctly and work nicely. You'll find the built application binary at `build/src/qlscribe`. Be aware that obviosly as time goes on my clanker-generated instructions will probably not be sufficient.\
I used:
- Arch Linux 
- GCC 16.2.1
- Qt 4.8.7

Btw I wrote this manually - least that I could do.