# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinxu.org>
# Contributor: Wael Nasreddine <gandalf@siemens-mobiles.org>
# Contributor: Tor Krill <tor@krill.nu>
# Contributor: Will Rea <sillywilly@gmail.com>
# Contributor: Valentine Sinitsyn <e_val@inbox.ru>

pkgbase=networkmanager
pkgname=(
  networkmanager
  libnm
  nm-cloud-setup
  networkmanager-docs
)
pkgver=1.46.0
pkgrel=2
pkgdesc="Network connection manager and user applications"
url="https://networkmanager.dev/"
arch=(x86_64)
license=(LGPL-2.1-or-later)
makedepends=(
  audit
  curl
  dhclient
  dhcpcd
  dnsmasq
  git
  glib2-docs
  gobject-introspection
  gtk-doc
  iproute2
  iptables
  iwd
  jansson
  libmm-glib
  libndp
  libnewt
  libpsl
  libteam
  meson
  modemmanager
  nftables
  nss
  openresolv
  pacrunner
  perl-yaml
  polkit
  ppp
  python-gobject
  systemd
  vala
  vala
  wpa_supplicant
)
checkdepends=(
  libx11
  python-dbus
)
source=(
  # Can't locate the public key (sfaye@redhat.com, E472337703D0C46002928B5790617850A125DE59)
  "git+https://gitlab.freedesktop.org/NetworkManager/NetworkManager.git#tag=$pkgver"
)
b2sums=('9285561e9c7ffb3e5d60ce60120d53e137c0ec2c36a373ee76befc3c93e54d191ff3187a287fbf05ab97bf8012ce72dd232a6756ebd0d59a5089c92b333a1bd2')

prepare() {
  cd NetworkManager
}

build() {
  local meson_options=(
    # build checks this option; injecting just via *FLAGS is broken
    -D b_lto=true

    # system paths
    -D dbus_conf_dir=/usr/share/dbus-1/system.d

    # platform
    -D dist_version="$pkgver-$pkgrel"
    -D session_tracking_consolekit=false
    -D suspend_resume=systemd
    -D modify_system=true
    -D selinux=false

    # features
    -D iwd=true
    -D teamdctl=true

    # configuration plugins
    -D config_plugins_default=keyfile
    -D ifupdown=false

    # handlers for resolv.conf
    -D netconfig=no
    -D config_dns_rc_manager_default=symlink

    # miscellaneous
    -D vapi=true
    -D docs=true
    -D more_asserts=no
    -D more_logging=false
    -D qt=false
  )

  arch-meson NetworkManager build "${meson_options[@]}"
  meson compile -C build
}

check() {
  NMTST_FORCE_REAL_ROOT=1 meson test -C build --print-errorlogs
}

_pick() {
  local p="$1" f d; shift
  for f; do
    d="$srcdir/$p/${f#$pkgdir/}"
    mkdir -p "$(dirname "$d")"
    mv "$f" "$d"
    rmdir -p --ignore-fail-on-non-empty "$(dirname "$f")"
  done
}

package_networkmanager() {
  depends=(
    audit
    curl
    iproute2
    jansson
    libmm-glib
    libndp
    libnewt
    libnm
    libpsl
    libteam
    mobile-broadband-provider-info
    wpa_supplicant
  )
  optdepends=(
    'bluez: Bluetooth support'
    'dhclient: alternative DHCP client'
    'dhcpcd: alternative DHCP client'
    'dnsmasq: connection sharing'
    'firewalld: firewall support'
    'iptables: connection sharing'
    'iwd: wpa_supplicant alternative'
    'modemmanager: cellular network support'
    'nftables: connection sharing'
    'openresolv: alternative resolv.conf manager'
    'pacrunner: PAC proxy support'
    'polkit: let non-root users control networking'
    'ppp: dialup connection support'
  )
  backup=(etc/NetworkManager/NetworkManager.conf)

  # NM wants to move to LGPL only, but there's still GPL code left
  license+=(GPL-2.0-or-later)

  meson install -C build --destdir "$pkgdir"

  cd "$pkgdir"

  # /etc/NetworkManager
  install -d etc/NetworkManager/{conf,dnsmasq}.d
  install -dm700 etc/NetworkManager/system-connections
  install -m644 /dev/stdin etc/NetworkManager/NetworkManager.conf <<END
# Configuration file for NetworkManager.
# See "man 5 NetworkManager.conf" for details.
END

  # packaged configuration
  install -Dm644 /dev/stdin usr/lib/NetworkManager/conf.d/20-connectivity.conf <<END
[connectivity]
uri=http://ping.archlinux.org/nm-check.txt
END

  shopt -s globstar

  _pick docs usr/share/gtk-doc

  _pick libnm usr/include/libnm
  _pick libnm usr/lib/girepository-1.0/NM-*
  _pick libnm usr/lib/libnm.*
  _pick libnm usr/lib/pkgconfig/libnm.pc
  _pick libnm usr/share/gir-1.0/NM-*
  _pick libnm usr/share/vala/vapi/libnm.*

  _pick cloud usr/lib/**/*nm-cloud-setup*
  _pick cloud usr/share/man/*/nm-cloud-setup*

  # Not actually packaged (https://bugs.archlinux.org/task/69138)
  _pick ovs usr/lib/systemd/system/NetworkManager.service.d/NetworkManager-ovs.conf

  # Restore empty dir
  install -d usr/lib/NetworkManager/dispatcher.d/no-wait.d
}

package_libnm() {
  pkgdesc="NetworkManager client library"
  depends=(
    glib2
    nss
    systemd-libs
    util-linux-libs
  )
  provides=(libnm.so)

  mv libnm/* "$pkgdir"
}

package_nm-cloud-setup() {
  pkgdesc="Automatically configure NetworkManager in cloud"
  depends=(networkmanager)

  mv cloud/* "$pkgdir"
}

package_networkmanager-docs() {
  pkgdesc+=" (API documentation)"
  depends=()

  mv docs/* "$pkgdir"
}

# vim:set sw=2 sts=-1 et:
