# E-Residency Manjaro

Disclaimer: Maybe there are better ways to do this, but I spend some hours figuring out what to do (Arch newbie). Anyway if it can help someone it's fine :)

## What ?

This repository contains the instructions to setup the Estonian e-card in Manjaro with cards issued after July 2019.
Remember your card is activated +-24 hours after is received.

## How ?

All types of electronic identification require installing the `ccid` package. After installation, enable, and start `pcscd.socket`. In addition, ACS smart cards also require the `acsccid` package. A useful set of tools is `pcsc-tools`.

Start `pcscd.socket`:

```shell
sudo systemctl start pcscd
sudo systemctl status pcscd
```

**Install DigiDoc4 and opensc-git**

Run (Example):

```shell
yay -S opensc-git
# if conflicts with opensc, remove opensc
# makepkg should be used later with opensc-git
```

Use this file:

```shell
# Maintainer: kevku <kevku@gmx.com>
pkgname=qdigidoc4
pkgver=4.2.2.51
pkgrel=1
pkgdesc="DigiDoc4 Client is an application for digitally signing and encrypting documents; the software includes functionality to manage Estonian ID-card - change pin codes, update certificates etc."
arch=('x86_64' 'i686')
url="http://www.id.ee/"
license=('LGPL2.1')
depends=('libdigidocpp>=3.13.9' 'opensc-git' 'qt5-svg' 'hicolor-icon-theme' 'desktop-file-utils')
makedepends=('cmake' 'qt5-tools' 'qt5-translations')
optdepends=('ccid: smart card support')
conflicts=('qdigidoc' 'qesteidutil')
provides=('qdigidoc' 'qesteidutil')
source=("https://installer.id.ee/media/ubuntu/pool/main/q/$pkgname/${pkgname}_$pkgver.orig.tar.xz")
sha256sums=('2a27523b3f98aeabc3cd8b135c5dc1e624acd97246be07b52a721c1ac3773de2')

prepare() {
  [[ -d "$pkgname-build" ]] && rm -r "$pkgname-build"
  mkdir "$pkgname-build"
  sed -i 's|#{ENV\["BUILD_NUMBER"\]}|51|g' cmake/modules/VersionInfo.cmake
}

build() {
  cd "$pkgname-build"
  cmake .. -DCMAKE_C_FLAGS:STRING="${CFLAGS} -ffile-prefix-map=$srcdir=." \
           -DCMAKE_CXX_FLAGS:STRING="${CXXFLAGS} -ffile-prefix-map=$srcdir=." \
           -DCMAKE_EXE_LINKER_FLAGS:STRING="${LDFLAGS}" \
           -DCMAKE_INSTALL_PREFIX="/usr" \
           -DCMAKE_INSTALL_LIBDIR="lib" \
           -DCMAKE_INSTALL_SYSCONFDIR="/etc"
  make
}

package() {
  cd "$pkgname-build"
  make DESTDIR="$pkgdir/" install
}
```

Run:

```shell
makepkg PKGBUILD
sudo mv ./pkg/qdigidoc4/usr/bin/qdigidoc4 /usr/bin
cd usr/bin && ./qdigidoc4
```


#### Chromium

Install chrome-token-signingAUR, enable the PIN 1 authentication in Google Chrome and Chromium by running the following command:

```shell
 modutil -dbdir sql:$HOME/.pki/nssdb -add opensc-pkcs11 -libfile onepin-opensc-pkcs11.so -mechanisms FRIENDLY
 ```

#### Firefox

To enable PIN 1 authentication in Firefox you should install esteidpkcs11loaderAUR and chrome-token-signingAUR. After restarting the browser make sure that "Firefox PKCS11 loader" extension is enabled.

For firefox-esr52AUR and other other Firefox forks you can use esteidfirefoxpluginAUR. 


