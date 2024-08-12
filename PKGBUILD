# Maintainer: Asuka Minato <i at asukaminato dot eu dot org>
# Maintainer: Kid <hi at xuann dot wang>
# Contributor: Jaime Martínez Rincón <jaime@jamezrin.name>

pkgname=notion-app-electron
pkgver=3.12.0
pkgrel=3
pkgdesc="Your connected workspace for wiki, docs & projects"
arch=(x86_64)
url=https://www.notion.so/desktop
license=(custom)
depends=(
	bash
	gcc-libs
	glibc
	hicolor-icon-theme
	electron31
)
makedepends=(
	p7zip
	asar
)
install=.install

source=(
	https://desktop-release.notion-static.com/Notion-${pkgver}.dmg
	https://github.com/WiseLibs/better-sqlite3/releases/download/v11.1.2/better-sqlite3-v11.1.2-electron-v125-linux-x64.tar.gz
	https://github.com/websockets/bufferutil/releases/download/v4.0.8/v4.0.8-linux-x64.tar
	notion-app
	notion.desktop
	notion.png
)
sha256sums=(
	67f646f6597af3c1e38fcb17684cfe243daae894bd970da5097612e74aa0f82d
	0147267cf7b33c77e0e8eb51de7d6bd6e24dae0c717f4d1a80efe743e2ab541e
	eac6fabcfa38e21c33763cb0e5efc1aa30c333cf9cc39f680fb8d12c88fefc93
	6db70ef54f01967980c41551ed03720f8c55d65506a07f1d37bffe47dae87c22
	87126c0da6f521ba93f9feefa5e1a96db66760bd63179a94feafbb0681496f1b
	61ecb0c334becf60da4a94482f10672434944e4d93e691651ec666cafb036646
)

prepare() {
	# extracting app.asar from dmg with 7z and ignoring header error
	7z x "$srcdir/Notion-${pkgver}.dmg" "Notion/Notion.app/Contents/Resources/app.asar" "Notion/Notion.app/Contents/Resources/app.asar.unpacked" -y -bse0 -bso0 || true
	# removing dmg already extracted
	rm "$srcdir/Notion-${pkgver}.dmg"
	# extracting resources from app.asar
	asar e "$srcdir/Notion/Notion.app/Contents/Resources/app.asar" "$srcdir/asar_patched"
	# replacing better_sqlite3 release in the patched resources
	mv "$srcdir/build/Release/better_sqlite3.node" "$srcdir/asar_patched/node_modules/better-sqlite3/build/Release/"
	# replacing bufferutil release in the patched resources
	mv "$srcdir/linux-x64/node.napi.node" "$srcdir/asar_patched/node_modules/bufferutil/build/Release/bufferutil.node"
	# removing some unnecessary files
	rm "$srcdir/asar_patched/node_modules/node-mac-window" -r
	rm "$srcdir/asar_patched/node_modules/better-sqlite3/build/Release/test_extension.node"
	rm $srcdir/asar_patched/*.provisionprofile
	rm "$srcdir/asar_patched/icon.icns"
	# adding tray icon to the unpacked resources
	cp "$srcdir/notion.png" "$srcdir/asar_patched/.webpack/main/trayIcon.png"
	# fully disabling auto updates
	sed -i 's/if("darwin"===process.platform){const e=s.systemPreferences?.getUserDefault(S,"boolean"),t=_.Store.getState().app.preferences?.isAutoUpdaterDisabled;return Boolean(e||t)}return!1/return!0/g' "$srcdir/asar_patched/.webpack/main/index.js"
	# disabling the app menu since most of the options won't work
	sed -i 's/Menu.setApplicationMenu(p(e))/Menu.setApplicationMenu(null)/g' "$srcdir/asar_patched/.webpack/main/index.js"
	# fixing tray icon and right click menu
	sed -i 's|this\.tray\.on("click",(()=>{this\.onClick()}))|this.tray.setContextMenu(this.trayMenu),this.tray.on("click",(()=>{this.onClick()}))|g' "$srcdir/asar_patched/.webpack/main/index.js"
	sed -i 's|getIcon(){[^}]*}|getIcon(){return s.default.join(__dirname, "trayIcon.png");}|g' "$srcdir/asar_patched/.webpack/main/index.js"
	# avoid running duplicated instances, restores the window if notion is running in tray
	sed -i 's/g=r(69340);/g=r(69340),appControl=r(21852),gotTheLock=l.app.requestSingleInstanceLock();/g' "$srcdir/asar_patched/.webpack/main/index.js"
	sed -i 's/l.app.on("render-process-gone",/!gotTheLock?l.app.quit():d.default.info("first instance!"),l.app.on("second-instance",()=>{const t=appControl.appController.getMostRecentlyFocusedWindowController();t?t.browserWindow.show():m.appController.newWindow({});}),l.app.on("render-process-gone",/g' "$srcdir/asar_patched/.webpack/main/index.js"
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
	install -Dm755 notion-app -t "$usr/bin"
	install -Dm644 "$srcdir/notion.desktop" -t "$share/applications"
	install -Dm644 "$srcdir/notion.png" -t "$share/icons/hicolor/256x256/apps"
	find "$pkgdir" -type d -empty -delete
}
