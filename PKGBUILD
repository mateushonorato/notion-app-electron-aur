# Maintainer: Asuka Minato <i at asukaminato dot eu dot org>
# Maintainer: Kid <hi at xuann dot wang>
# Contributor: Jaime Martínez Rincón <jaime@jamezrin.name>

pkgname=notion-app-electron
pkgver=3.3.0
pkgrel=2
pkgdesc='Your connected workspace for wiki, docs & projects'
arch=(x86_64)
url=https://www.notion.so/desktop
license=(custom)
depends=(
	bash
	gcc-libs
	glibc
	hicolor-icon-theme
	electron28
)
makedepends=(
	p7zip
	asar
	icoutils
)
install=.install

source=(
	"https://desktop-release.notion-static.com/Notion%20Setup%20${pkgver}.exe"
	https://github.com/WiseLibs/better-sqlite3/releases/download/v9.4.3/better-sqlite3-v9.4.3-electron-v119-linux-x64.tar.gz
	notion-app
	notion.desktop
)
sha256sums=('7c04c8248f186b5a51a9871db613f0cc8b53a04ad6fadd7fd40376afa8aaab35'
            '4717969345fd961dee09fb9813e25e4e5b51f284cf1d3ac0eb6dd855fbce39e9'
            '1ee5a3847a6aba3cba9b757ddc0f2364a5c24c4b63e057e03aede5c56f126642'
            '19a5f973f1e9291081aa05512e07c61447e8c30e1a43dd22d0cc1090837d1e19')

prepare() {
	# bsdtar can't recognize 3.2.1, so use 7z
	7z x ./*.exe
	rm ./*.exe
	7z x ./**/app-64.7z
	rm ./**/app-64.7z
	asar e "$srcdir"/**/app.asar "$srcdir/unpacked"
	icotool -x -w 256 "$srcdir/unpacked/icon.ico" -o "$srcdir/notion.png"
	icotool -x -w 256 "$srcdir/resources/trayIcon.ico" -o "$srcdir/trayIcon.png"

	sed -i -e 's/"win32"===process.platform/(true)/g
		    s/_.Store.getState().app.preferences?.isAutoUpdaterDisabled/(true)/g
		    s!extra-resources!/usr/share/notion-app!g
		    s/trayIcon.ico/trayIcon.png/g' "$srcdir/unpacked/.webpack/main/index.js"
	find $srcdir \( -name "clang-format.js" -or -name "conversion.js" -or -name "eslint-format.js" \) -delete -printf "rm %p to make namcap happy.\n"
}

package() {
	local usr="$pkgdir/usr"
	local share="$usr/share"
	local lib="$usr/lib/notion-app"

	install -d "$lib"
	cp -a "$srcdir/unpacked/"{package.json,node_modules,.webpack} "$lib"
	install -Dm644 "$srcdir/build/Release/better_sqlite3.node" -t "$lib/node_modules/better-sqlite3/build/Release/"
	install -Dm755 notion-app -t "$usr/bin"
	install -Dm644 "$srcdir/notion.desktop" -t "$share/applications"
	install -Dm644 "$srcdir/notion.png" -t "$share/icons/hicolor/256x256/apps"
	install -Dm644 "$srcdir/trayIcon.png" -t "$share/notion-app"
	find "$pkgdir" -type d -empty -delete
}
