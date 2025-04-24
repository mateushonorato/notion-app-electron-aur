# Maintainer: Asuka Minato <i at asukaminato dot eu dot org>
# Maintainer: Kid <hi at xuann dot wang>
# Maintainer: Mateus Honorato <mateush.honorato@gmail.com>
# Contributor: Jaime Martínez Rincón <jaime@jamezrin.name>

pkgname=notion-app-electron
pkgver=4.9.1
_bettersqlite3ver=11.9.1
_bufferutilver=4.0.9
_elecronver=132
pkgrel=1
pkgdesc="Your connected workspace for wiki, docs & projects"
arch=(x86_64)
url=https://www.notion.so/desktop
license=(custom)
depends=(
	bash
	gcc-libs
	glibc
	hicolor-icon-theme
	electron34
)
makedepends=(
	p7zip
	asar
)
install=.install

source=(
	https://desktop-release.notion-static.com/Notion%20Setup%20${pkgver}.exe
	https://github.com/WiseLibs/better-sqlite3/releases/download/v${_bettersqlite3ver}/better-sqlite3-v${_bettersqlite3ver}-electron-v${_elecronver}-linux-x64.tar.gz
	https://github.com/websockets/bufferutil/releases/download/v${_bufferutilver}/v${_bufferutilver}-linux-x64.tar
	notion-app
	notion.desktop
	notion.png
)
sha256sums=('9dcf5624d3688f87d95ef3e85c1b0ec7669f0b764395c172db447ff44c473360'
            'a976d128d113e6a6ed3b5505371a7269e0871375361480e9fc9ebf0ccedc5f59'
            '2139aae79c5a4fd4d07467bd9b7872ea109483aa43b3dfd6c8d3725ccba009be'
            '8b38d2bb062799d90dee8d28275aa8bea27aa25a57024aefb3346ff93c803623'
            '19a5f973f1e9291081aa05512e07c61447e8c30e1a43dd22d0cc1090837d1e19'
            '61ecb0c334becf60da4a94482f10672434944e4d93e691651ec666cafb036646')

options=(!emptydirs)

prepare() {
	# extracting app.asar from installer with 7z and ignoring errors
	7z x "./Notion%20Setup%20${pkgver}.exe" "\$PLUGINSDIR/app-64.7z" -y -bse0 -bso0 || true
	7z x "./\$PLUGINSDIR/app-64.7z" "resources/app.asar" "resources/app.asar.unpacked" -y -bse0 -bso0 || true
	rm "./Notion%20Setup%20${pkgver}.exe"
	rm "./\$PLUGINSDIR/app-64.7z"
	# extracting resources from app.asar
	asar e "$srcdir/resources/app.asar" "$srcdir/asar_patched"
	# replacing better_sqlite3 release in the patched resources
	install -vDm644 "$srcdir/build/Release/better_sqlite3.node" -t "$srcdir/asar_patched/node_modules/better-sqlite3/build/Release/"
	# replacing bufferutil release in the patched resources
	install -vDm644 "$srcdir/linux-x64/node.napi.node" "$srcdir/asar_patched/node_modules/bufferutil/build/Release/bufferutil.node"
	# removing some unnecessary files (keeping them in this version to see if it improves stability)
	# rm "$srcdir/asar_patched/node_modules/node-mac-window" -r
	# rm "$srcdir/asar_patched/node_modules/better-sqlite3/build/Release/test_extension.node"
	# adding tray icon to the unpacked resources
	install -vDm644 "$srcdir/notion.png" "$srcdir/asar_patched/.webpack/main/trayIcon.png"
	# fixing tray icon and right click menu
	sed -i 's|this.tray.on("click",(()=>{this.onClick()}))|this.tray.setContextMenu(this.trayMenu),this.tray.on("click",(()=>{this.onClick()}))|g' "$srcdir/asar_patched/.webpack/main/index.js"
	sed -i 's|getIcon(){[^}]*}|getIcon(){return require("path").resolve(__dirname, "trayIcon.png");}|g' "$srcdir/asar_patched/.webpack/main/index.js"
	# fake the useragent as windows to fix the spellchecker languages selector and other issues
	sed -i 's|e.setUserAgent(`${e.getUserAgent()} WantsServiceWorker`),|e.setUserAgent(`${e.getUserAgent().replace("Linux", "Windows")} WantsServiceWorker`),|g' "$srcdir/asar_patched/.webpack/main/index.js"
	# fully disabling auto updates
	sed -i 's|if("darwin"===process.platform){const e=l.systemPreferences?.getUserDefault(C,"boolean"),t=M.Store.getState().app.preferences?.isAutoUpdaterDisabled,r=M.Store.getState().app.preferences?.isAutoUpdaterOSSupportBypass,n=(0,y.isOsUnsupportedForAutoUpdates)();return Boolean(e\|\|t\|\|!r&&n)}return!1|return!0|g' "$srcdir/asar_patched/.webpack/main/index.js"
	# avoid running duplicated instances, fixes url opening
	sed -i 's|handleOpenUrl);else if("win32"===process.platform)|handleOpenUrl);else if("linux"===process.platform)|g' "$srcdir/asar_patched/.webpack/main/index.js"
	sed -i 's|async function(){await(0,m.setupObservability)(),|o.app.requestSingleInstanceLock() ? async function(){await(0,m.setupObservability)(),|g' "$srcdir/asar_patched/.webpack/main/index.js"
	sed -i 's|setupAboutPanel)()}()}()|setupAboutPanel)()}()}() : o.app.quit();|g' "$srcdir/asar_patched/.webpack/main/index.js"
	# use the windows version of the tray menu
	sed -i 's|r="win32"===process.platform?function(e,t)|r="linux"===process.platform?function(e,t)|g' "$srcdir/asar_patched/.webpack/main/index.js"
	# this can disable app menu when the options won't work. disbled in the current version because it's working now, but it's here for future reference
	# sed -i 's|Menu.setApplicationMenu(p(e))|Menu.setApplicationMenu(null)|g' "$srcdir/asar_patched/.webpack/main/index.js"
	# repacking asar with all the patches
	asar p "$srcdir/asar_patched" "$srcdir/app.asar" --unpack *.node
}

package() {
	local usr="$pkgdir/usr"
	local share="$usr/share"
	local lib="$usr/lib/notion-app"

	install -d "$lib"
	cp "$srcdir/app.asar" "$lib"
	cp "$srcdir/app.asar.unpacked" "$lib" -r
	install -vDm755 notion-app -t "$usr/bin"
	install -vDm644 "$srcdir/notion.desktop" -t "$share/applications"
	install -vDm644 "$srcdir/notion.png" -t "$share/icons/hicolor/256x256/apps"
}
