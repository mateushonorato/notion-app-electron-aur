# Maintainer: Asuka Minato <i at asukaminato dot eu dot org>
# Maintainer: Kid <hi at xuann dot wang>
# Contributor: Jaime Martínez Rincón <jaime@jamezrin.name>

pkgname=notion-app-electron
pkgver=3.13.0
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
	electron32
)
makedepends=(
	p7zip
	asar
)
install=.install

source=(
	https://desktop-release.notion-static.com/Notion-${pkgver}.dmg
	https://github.com/WiseLibs/better-sqlite3/releases/download/v11.2.1/better-sqlite3-v11.2.1-electron-v128-linux-x64.tar.gz
	https://github.com/websockets/bufferutil/releases/download/v4.0.8/v4.0.8-linux-x64.tar
	notion-app
	notion.desktop
	notion.png
)
sha256sums=(
	6f39295526f524a3bf117df13a4c0a01b21e8328e2aff428093382b49a26f4b8
	bd96747ea2bbaf15018d4a5a7fabaa012740ea77b3e415573266d66b370372d4
	eac6fabcfa38e21c33763cb0e5efc1aa30c333cf9cc39f680fb8d12c88fefc93
	19b4eb889ab9ef429f7537f7d25da7c49710114a397ccff0a5bc943db53651f8
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
	sed -i 's/if("darwin"===process.platform){const e=s.systemPreferences?.getUserDefault(E,"boolean"),t=_.Store.getState().app.preferences?.isAutoUpdaterDisabled;return Boolean(e||t)}return!1/return!0/g' "$srcdir/asar_patched/.webpack/main/index.js"
	# disabling the app menu since most of the options won't work
	sed -i 's/Menu.setApplicationMenu(p(e))/Menu.setApplicationMenu(null)/g' "$srcdir/asar_patched/.webpack/main/index.js"
	# fixing tray icon and right click menu
	sed -i 's|this\.tray\.on("click",(()=>{this\.onClick()}))|this.tray.setContextMenu(this.trayMenu),this.tray.on("click",(()=>{this.onClick()}))|g' "$srcdir/asar_patched/.webpack/main/index.js"
	sed -i 's|getIcon(){[^}]*}|getIcon(){return s.default.join(__dirname, "trayIcon.png");}|g' "$srcdir/asar_patched/.webpack/main/index.js"
	# avoid running duplicated instances, restores the window if notion is running in tray
	sed -i 's/v=r(69340),/v=r(69340),appControl=r(21852),gotTheLock=d.app.requestSingleInstanceLock(),/g' "$srcdir/asar_patched/.webpack/main/index.js"
	sed -i 's/d.app.on("render-process-gone",/!gotTheLock?d.app.quit():t.logger.info("first instance!"),d.app.on("second-instance",()=>{const u=appControl.appController.getMostRecentlyFocusedWindowController();if(u) u.browserWindow.show();}),d.app.on("render-process-gone",/g' "$srcdir/asar_patched/.webpack/main/index.js"
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
