# Maintainer: Aaron Graves <linux@ajgraves.com>
# Contributor: Julian Raufelder <arch@raufelder.com>
# Contributor: Morten Linderud <morten@linderud.pw>
# Contributor: Sebastian Stenzel <sebastian.stenzel@gmail.com>

pkgname=cryptomator
pkgver=1.13.0
pkgrel=2
pkgdesc="Multiplatform transparent client-side encryption of your files in the cloud."
arch=('any')
url="https://cryptomator.org/"
license=('GPL3')
depends=('fuse3' 'alsa-lib' 'hicolor-icon-theme' 'libxtst' 'libnet' 'libxrender')
makedepends=('maven' 'unzip')
optdepends=('keepassxc-cryptomator: Use KeePassXC to store vault passwords' 'ttf-hanazono: Install this font when using Japanese system language')
source=("cryptomator-${pkgver//_/-}.tar.gz::https://github.com/cryptomator/cryptomator/archive/refs/tags/${pkgver//_/-}.tar.gz"
        "cryptomator-${pkgver//_/-}.tar.gz.asc::https://github.com/cryptomator/cryptomator/releases/download/${pkgver//_/-}/cryptomator-${pkgver//_/-}.tar.gz.asc")
source_x86_64=("jdk.tar.gz::https://github.com/adoptium/temurin22-binaries/releases/download/jdk-22.0.1%2B8/OpenJDK22U-jdk_x64_linux_hotspot_22.0.1_8.tar.gz"
               "openjfx.zip::https://download2.gluonhq.com/openjfx/22.0.1/openjfx-22.0.1_linux-x64_bin-jmods.zip")
source_aarch64=("jdk.tar.gz::https://github.com/adoptium/temurin22-binaries/releases/download/jdk-22.0.1%2B8/OpenJDK22U-jdk_aarch64_linux_hotspot_22.0.1_8.tar.gz"
                "openjfx.zip::https://download2.gluonhq.com/openjfx/22.0.1/openjfx-22.0.1_linux-aarch64_bin-jmods.zip")
noextract=('jdk.tar.gz' 'openjfx.zip')
sha256sums=('d7d0389c3b8632d5fec08a38973c305ee48bcdd2208861f74f0fe570bf11ce30'
            'SKIP')
sha256sums_x86_64=('e59c6bf801cc023a1ea78eceb5e6756277f1564cd0a421ea984efe6cb96cfcf8'
                   'fbb22f35951c2e049cc2554dd03c2c56b4f5adc4b2ae9248872f46175ac103d8')
sha256sums_aarch64=('d8488fa1e4e8c1e318cef4c0fc3842a7f15a4cf52b27054663bb94471f54b3fa'
                    '1982ad168a5e8d7cf4a9458a7d088b4f0552d0ac3f24f23fb88f8bc7e8d69a13')
options=('!strip')

validpgpkeys=('58117AFA1F85B3EEC154677D615D449FE6E6A235')

build() {
  export JAVA_HOME="${srcdir}/jdk"
  export JMODS_PATH="${srcdir}/jmods:${JAVA_HOME}/jmods"

  tar xvfz jdk.tar.gz --transform 's!^[^/]*!jdk!'

  mkdir jmods
  unzip -j openjfx.zip \*/javafx.base.jmod \*/javafx.controls.jmod \*/javafx.fxml.jmod \*/javafx.graphics.jmod -d jmods

  cd "${srcdir}/cryptomator-${pkgver//_/-}"

  mvn -B clean package -Djavafx.platform=linux -DskipTests -Plinux

  cp LICENSE.txt target
  cp target/cryptomator-*.jar target/mods

  cd target

  "$JAVA_HOME/bin/jlink" \
    --output runtime \
    --module-path "$JMODS_PATH" \
    --add-modules java.base,java.desktop,java.instrument,java.logging,java.naming,java.net.http,java.scripting,java.sql,java.xml,javafx.base,javafx.graphics,javafx.controls,javafx.fxml,jdk.unsupported,jdk.security.auth,jdk.accessibility,jdk.management.jfr,jdk.net \
    --strip-native-commands \
    --no-header-files \
    --no-man-pages \
    --strip-debug \
    --compress=zip-0

  ##Note: jpackage does not allow -beta suffixes, have to strip those
  "$JAVA_HOME/bin/jpackage" \
    --type app-image \
    --runtime-image runtime \
    --input libs \
    --module-path mods \
    --module org.cryptomator.desktop/org.cryptomator.launcher.Cryptomator \
    --dest . \
    --name cryptomator \
    --vendor "Skymatic GmbH" \
    --java-options "--enable-preview" \
    --java-options '--enable-native-access=org.cryptomator.jfuse.linux.amd64,org.cryptomator.jfuse.linux.aarch64,org.purejava.appindicator' \
    --copyright "(C) 2016 - 2024 Skymatic GmbH" \
    --java-options "-Xss5m" \
    --java-options "-Xmx256m" \
    --java-options "-Dfile.encoding=\"utf-8\"" \
    --java-options "-Djava.net.useSystemProxies=true" \
    --java-options "-Dcryptomator.logDir=\"@{userhome}/.local/share/Cryptomator/logs\"" \
    --java-options "-Dcryptomator.pluginDir=\"@{userhome}/.local/share/Cryptomator/plugins\"" \
    --java-options "-Dcryptomator.settingsPath=\"@{userhome}/.config/Cryptomator/settings.json:~/.Cryptomator/settings.json\"" \
    --java-options "-Dcryptomator.p12Path=\"@{userhome}/.config/Cryptomator/key.p12\"" \
    --java-options "-Dcryptomator.ipcSocketPath=\"@{userhome}/.config/Cryptomator/ipc.socket\"" \
    --java-options "-Dcryptomator.mountPointsDir=\"@{userhome}/.local/share/Cryptomator/mnt\"" \
    --java-options "-Dcryptomator.showTrayIcon=true" \
    --java-options "-Dcryptomator.disableUpdateCheck=true" \
    --java-options "-Dcryptomator.buildNumber=\"aur-${pkgrel}\"" \
    --java-options "-Dcryptomator.appVersion=\"${pkgver//_/-}\"" \
    --app-version "${pkgver//_*/}" \
    --verbose
}

package() {
  install -Dm644 "${srcdir}/cryptomator-${pkgver//_/-}/dist/linux/common/application-vnd.cryptomator.vault.xml" "${pkgdir}/usr/share/mime/packages/cryptomator-vault.xml"
  install -Dm644 "${srcdir}/cryptomator-${pkgver//_/-}/dist/linux/common/org.cryptomator.Cryptomator.desktop" "${pkgdir}/usr/share/applications/org.cryptomator.Cryptomator.desktop"
  install -Dm644 "${srcdir}/cryptomator-${pkgver//_/-}/dist/linux/common/org.cryptomator.Cryptomator256.png" "${pkgdir}/usr/share/icons/hicolor/256x256/apps/org.cryptomator.Cryptomator.png"
  install -Dm644 "${srcdir}/cryptomator-${pkgver//_/-}/dist/linux/common/org.cryptomator.Cryptomator512.png" "${pkgdir}/usr/share/icons/hicolor/512x512/apps/org.cryptomator.Cryptomator.png"
  install -Dm644 "${srcdir}/cryptomator-${pkgver//_/-}/dist/linux/common/org.cryptomator.Cryptomator.svg" "${pkgdir}/usr/share/icons/hicolor/scalable/apps/org.cryptomator.Cryptomator.svg"
  install -Dm644 "${srcdir}/cryptomator-${pkgver//_/-}/dist/linux/common/org.cryptomator.Cryptomator.tray.svg" "${pkgdir}/usr/share/icons/hicolor/scalable/apps/org.cryptomator.Cryptomator.tray.svg"
  install -Dm644 "${srcdir}/cryptomator-${pkgver//_/-}/dist/linux/common/org.cryptomator.Cryptomator.tray-unlocked.svg" "${pkgdir}/usr/share/icons/hicolor/scalable/apps/org.cryptomator.Cryptomator.tray-unlocked.svg"
  install -Dm644 "${srcdir}/cryptomator-${pkgver//_/-}/dist/linux/common/org.cryptomator.Cryptomator.tray.svg" "${pkgdir}/usr/share/icons/hicolor/symbolic/apps/org.cryptomator.Cryptomator.tray-symbolic.svg"
  install -Dm644 "${srcdir}/cryptomator-${pkgver//_/-}/dist/linux/common/org.cryptomator.Cryptomator.tray-unlocked.svg" "${pkgdir}/usr/share/icons/hicolor/symbolic/apps/org.cryptomator.Cryptomator.tray-unlocked-symbolic.svg"

  mkdir -p "${pkgdir}/opt/cryptomator/"
  cp -R "${srcdir}/cryptomator-${pkgver//_/-}/target/cryptomator" "${pkgdir}/opt/"
  install -Dm644 "${srcdir}/cryptomator-${pkgver//_/-}/target/LICENSE.txt" -t "${pkgdir}/usr/share/licenses/${pkgname}"

  mkdir -p "${pkgdir}/usr/bin"
  ln -s "/opt/cryptomator/bin/cryptomator" "${pkgdir}/usr/bin/cryptomator"
}
