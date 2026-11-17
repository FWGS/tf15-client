# TF15-Client [![Build Status](https://github.com/FWGS/tf15-client/actions/workflows/build.yml/badge.svg)](https://github.com/FWGS/tf15-client/actions) <img align="right" width="128" height="128" src="https://github.com/FWGS/tf15-client/raw/master/android/app/src/main/ic_launcher-playstore.png" alt="TF15-Client" />
Team Fortress Classic reverse-engineered.

## Donate

If you enjoy our work, consider supporting the contributors through the links below.

| Contributor | Support |
|:--|:--:|
| [Velaron](https://github.com/Velaron) | [![Boosty](https://img.shields.io/badge/Boosty-F15F2C?logo=boosty&logoColor=fff&style=for-the-badge)](https://boosty.to/velaron) [![Buy Me a Coffee](https://img.shields.io/badge/Buy%20Me%20a%20Coffee-FFDD00?logo=buymeacoffee&logoColor=000&style=for-the-badge)](https://www.buymeacoffee.com/velaron) |
| [Vladislav4KZ](https://github.com/Vladislav4KZ) | [![Boosty](https://img.shields.io/badge/Boosty-F15F2C?logo=boosty&logoColor=fff&style=for-the-badge)](https://boosty.to/rasstaman1337) [![PayPal](https://img.shields.io/badge/PayPal-003087?logo=paypal&logoColor=fff&style=for-the-badge)](https://paypal.me/rasstaman1337) |

## Download
You can download a build at the [Releases](https://github.com/FWGS/tf15-client/releases/tag/continuous) section, or use these links:
* [Android](https://github.com/FWGS/tf15-client/releases/download/continuous/tf15-client.apk)
* [Linux](https://github.com/FWGS/tf15-client/releases/download/continuous/tf15-client_linux.tar.gz) (not recommended, just use the Steam version)
* [Windows](https://github.com/FWGS/tf15-client/releases/download/continuous/tf15-client_win32.zip) (not recommended, same as above)

## Installation
To run TF15-Client you need the latest developer build of Xash3D FWGS, which you can get [here](https://github.com/FWGS/xash3d-fwgs/releases/tag/continuous).
You have to own the [game on Steam](https://store.steampowered.com/app/20/Team_Fortress_Classic/) and copy `valve` and `tfc` folders into your Xash3D FWGS directory.
After that, just install the APK and run.

## Building
Clone the source code:
```
git clone https://github.com/FWGS/tf15-client --recursive
```
### Windows
```
cmake -A Win32 -S . -B build
cmake --build build --config Release
```
### Linux
```
cmake -S . -B build
cmake --build build --config Release
```
### Android
```
cd android
./gradlew assembleRelease
```
