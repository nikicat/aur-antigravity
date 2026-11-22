# Maintainer: Nikolay Bryskin (fork with self-updating pkgver)
# Contributor: AlphaLynx <alphalynx at alphalynx dot dev>

pkgname=antigravity
pkgver=0  # placeholder: pkgver() sets the real one
pkgrel=1
pkgdesc='Google Antigravity 2.0 multi-agent orchestration platform'
arch=(aarch64 x86_64)
url=https://antigravity.google/product/antigravity-2
license=(LicenseRef-Google-Antigravity)
depends=(alsa-lib
         at-spi2-core
         cairo
         dbus
         expat
         glib2
         glibc
         gtk3
         libcups
         libgcc
         libx11
         libxcb
         libxcomposite
         libxdamage
         libxext
         libxfixes
         libxkbcommon
         libxrandr
         mesa
         nspr
         nss
         pango
         systemd-libs)
options=(!strip !debug)
source=(antigravity.desktop
        antigravity.png)
b2sums=('ec2c74f0e6f2458f2ef1c67b1321c058a3f7c422e364d8871a0a933220ec52ee5736e449cb797b6f135cff1f4adfe6f81e0fdbc76d3ec67c245aea002b8577d8'
        '772849ebf4574dccc7d2fc30751baed10b4a5d4091c524f3d69938192387693501c0c54ea5b43fa2ed382f70ecdff369a4c65ce3d0f778c68712fa0c2b5e84b8')
makedepends=(curl jq)
# The tarball is not in source=(): prepare() downloads the latest release the
# updater lists and records its version; pkgver() reports that version, or asks
# the updater when prepare() was skipped (the zxcaur update check).
# makepkg runs prepare() before pkgver(), so prepare() must not use $pkgver.
# Google ships no checksum (GCS gives only crc32c), so integrity rests on TLS.
_updater=https://antigravity-hub-auto-updater-974169037036.us-central1.run.app

pkgver() {
    cat "$srcdir/.version" 2>/dev/null || curl -fsSL "$_updater/releases" | jq -er '.[0].version'
}

prepare() {
    local _ver _id _plat _tgz
    read -r _ver _id < <(curl -fsSL "$_updater/releases" | jq -er '.[0] | "\(.version) \(.execution_id)"')
    # both end up in paths and a URL: allow only plain versions and numeric ids
    [[ $_ver =~ ^[0-9][0-9.]*$ && $_id =~ ^[0-9]+$ ]] || { echo "Bad updater entry: $_ver $_id"; return 1; }
    case $CARCH in x86_64) _plat=linux-x64 ;; aarch64) _plat=linux-arm ;; esac
    # build id in the name: Google has re-cut builds without changing the version
    _tgz="${SRCDEST:-$startdir}/Antigravity-$_ver-$_id-$CARCH.tar.gz"
    if [[ ! -s $_tgz ]]; then
        curl -fL -o "$_tgz.part" \
            "https://storage.googleapis.com/antigravity-public/antigravity-hub/$_ver-$_id/$_plat/Antigravity.tar.gz"
        mv "$_tgz.part" "$_tgz"
    fi
    rm -rf Antigravity-*/
    bsdtar -xf "$_tgz" -C "$srcdir"
    echo "$_ver" > "$srcdir/.version"
}

package() {
    install -d "$pkgdir/opt"
    cp -a Antigravity-*/ "$pkgdir/opt/Antigravity"

    install -d "$pkgdir/usr/bin"
    ln -s /opt/Antigravity/antigravity "$pkgdir/usr/bin/$pkgname"

    install -d "$pkgdir/usr/share/licenses/$pkgname"
    ln -s /opt/Antigravity/LICENSE.electron.txt -t "$pkgdir/usr/share/licenses/$pkgname"
    ln -s /opt/Antigravity/LICENSES.chromium.html -t "$pkgdir/usr/share/licenses/$pkgname"

    install -Dm644 $pkgname.desktop -t "$pkgdir/usr/share/applications"
    install -Dm644 $pkgname.png -t "$pkgdir/usr/share/pixmaps"
}
