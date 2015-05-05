# Maintainer: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# Maintainer: Jan de Groot <jgc@archlinxu.org>
# Contributor: Wael Nasreddine <gandalf@siemens-mobiles.org>
# Contributor: Tor Krill <tor@krill.nu>
# Contributor: Will Rea <sillywilly@gmail.com>
# Contributor: Valentine Sinitsyn <e_val@inbox.ru>

pkgbase=networkmanager
pkgname=(networkmanager libnm-glib)
pkgver=1.0.2
pkgrel=1
pkgdesc="Network Management daemon"
arch=(i686 x86_64)
license=(GPL2 LGPL2.1)
url="http://www.gnome.org/projects/NetworkManager/"
_pppver=2.4.7
makedepends=(intltool dhclient iptables gobject-introspection gtk-doc "ppp=$_pppver"
             modemmanager dbus-glib iproute2 libnl nss polkit wpa_supplicant libsoup
             systemd libmm-glib rp-pppoe libnewt libndp libteam vala)
checkdepends=(libx11 python-gobject python-dbus)
#source=(git://anongit.freedesktop.org/NetworkManager/NetworkManager#commit=93c1041
source=(http://ftp.gnome.org/pub/gnome/sources/NetworkManager/${pkgver:0:3}/NetworkManager-$pkgver.tar.xz
        NetworkManager.conf disable_set_hostname.patch)
sha256sums=('359385707494bedbb48cfe0992ccfbcc4ac147dae1f7a47055c71e96439508ff'
            '759db295ddae7a6dc6b29211fc0ec08695f875584d456dd146d3679e2c33e2e3'
            '25056837ea92e559f09563ed817e3e0cd9333be861b8914e45f62ceaae2e0460')

prepare() {
  cd NetworkManager-$pkgver
  patch -Np1 -i ../disable_set_hostname.patch
  NOCONFIGURE=1 ./autogen.sh
}

build() {
  cd NetworkManager-$pkgver
  ./configure --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var \
    --sbindir=/usr/bin \
    --libexecdir=/usr/lib/networkmanager \
    --with-crypto=nss \
    --with-dhclient=/usr/bin/dhclient \
    --without-dhcpcd \
    --with-dnsmasq=/usr/bin/dnsmasq \
    --with-iptables=/usr/bin/iptables \
    --with-systemdsystemunitdir=/usr/lib/systemd/system \
    --with-udev-dir=/usr/lib/udev \
    --with-resolvconf=/usr/bin/resolvconf \
    --with-pppd=/usr/bin/pppd \
    --with-pppd-plugin-dir=/usr/lib/pppd/$_pppver \
    --with-pppoe=/usr/bin/pppoe \
    --with-kernel-firmware-dir=/usr/lib/firmware \
    --with-session-tracking=systemd \
    --with-modem-manager-1 \
    --disable-static \
    --enable-more-warnings=no \
    --disable-wimax \
    --enable-modify-system \
    --enable-doc

  make
}

check() {
  cd NetworkManager-$pkgver
  make -k check
}

package_networkmanager() {
  depends=(libnm-glib iproute2 libnl polkit wpa_supplicant dhclient libsoup libmm-glib libnewt
           libndp libteam)
  optdepends=('dnsmasq: connection sharing'
              'bluez: Bluetooth support'
              'openresolv: resolvconf support'
              'ppp: dialup connection support'
              'rp-pppoe: ADSL support'
              'modemmanager: cellular network support')
  install=networkmanager.install
  backup=('etc/NetworkManager/NetworkManager.conf')

  cd NetworkManager-$pkgver
  make DESTDIR="$pkgdir" install
  make DESTDIR="$pkgdir" -C libnm uninstall
  make DESTDIR="$pkgdir" -C libnm-glib uninstall
  make DESTDIR="$pkgdir" -C libnm-util uninstall
  make DESTDIR="$pkgdir" -C vapi uninstall

  # Some stuff to move is left over
  mv "$pkgdir/usr/include" ..
  mv "$pkgdir/usr/lib/pkgconfig" ..

  install -m644 ../NetworkManager.conf "$pkgdir/etc/NetworkManager/"
  install -m755 -d "$pkgdir/etc/NetworkManager/dnsmasq.d"

  rm -r "$pkgdir/var/run"
}

package_libnm-glib() {
  pkgdesc="NetworkManager library"
  depends=(libsystemd nss dbus-glib libutil-linux)

  install -d "$pkgdir/usr/lib"
  mv include "$pkgdir/usr"
  mv pkgconfig "$pkgdir/usr/lib"

  cd NetworkManager-$pkgver
  make DESTDIR="$pkgdir" -C libnm install
  make DESTDIR="$pkgdir" -C libnm-util install
  make DESTDIR="$pkgdir" -C libnm-glib install
  make DESTDIR="$pkgdir" -C vapi install
}
