# Maintainer: Jan de Groot <jgc@archlinxu.org>
# Contri-butor: Wael Nasreddine <gandalf@siemens-mobiles.org>
# Contributor: Tor Krill <tor@krill.nu>
# Contributor: Will Rea <sillywilly@gmail.com>
# Contributor: Valentine Sinitsyn <e_val@inbox.ru>

pkgname=networkmanager
pkgver=0.8.0.998
pkgrel=1
pkgdesc="Network Management daemon"
arch=('i686' 'x86_64')
license=('GPL')
url="http://www.gnome.org/projects/NetworkManager/"
depends=('wireless_tools' 'iproute2' 'libnl>=1.1' 'ppp' 'dhcpcd>=5.2.2' 'wpa_supplicant>=0.6.10' 'iptables' 'nss>=3.12.4' 'polkit>=0.96' 'udev>=151')
makedepends=('pkgconfig' 'intltool')
optdepends=('modemmanager: for modem management service')
options=('!libtool' '!makeflags')
backup=('etc/NetworkManager/nm-system-settings.conf')
replaces=('libnetworkmanager')
provides=("libnetworkmanager=${pkgver}")
conflicts=('libnetworkmanager')
source=(http://ftp.gnome.org/pub/gnome/sources/NetworkManager/0.8/NetworkManager-${pkgver}.tar.bz2
        nm-system-settings.conf
        disable_set_hostname.patch)
sha256sums=('6ae86be2bbeb352160d1741b494705eae9c6b971e53b5b865af1cb379126e2ca'
            '44b048804c7c0b8b3b0c29b8632b6ad613c397d0a1635ec918e10c0fbcdadf21'
            'aee905e15ba6fae154de05aaa1af9fbdd2855d66cdfbbcc90cfc0c8e98cdfda6')

build() {
  cd "${srcdir}/NetworkManager-${pkgver}"
  patch -Np1 -i "${srcdir}/disable_set_hostname.patch" || return 1

  ./configure --prefix=/usr --sysconfdir=/etc \
      --with-distro=arch --localstatedir=/var \
      --libexecdir=/usr/lib/networkmanager \
      --disable-static --with-dhcpcd=/sbin/dhcpcd \
      --with-crypto=nss --with-iptables=/usr/sbin/iptables \
      --enable-more-warnings=no || return 1

  make || return 1
  make DESTDIR="${pkgdir}" install || return 1
  install -m644 "${srcdir}/nm-system-settings.conf" "${pkgdir}/etc/NetworkManager/" || return 1
}
