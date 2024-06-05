# Maintainer: Luke Arms <luke@arms.to>
# Contributor: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
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
pkgrel=3
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
_commit=e39f48a30a2ef7b445276a859bbd5255e4c5071d  # tags/1.46.0^0
source=(
  "git+https://gitlab.freedesktop.org/NetworkManager/NetworkManager.git#commit=$_commit"
  0001-fix-ppp-noipv6.patch
  0002-fix-ppp-disable-ipv6.patch
)
b2sums=('SKIP'
        '1e4f6c52abd11d511e5d58a2d25935001d61d75451e35be52a65964ddfb054c70b5e5151970b18b0510962583e0b0c3f050c89a6a829c4e515ac367b71dc16c6'
        '6c570f25fac1a67d73dd9d8bd8fef4fb29e4b4044a13c833a029f0c878b15847061caf5d3c67b1a447a48aaa3e5f16562116a6a44b0004ce14f8b4fc583da055')

pkgver() {
  cd NetworkManager
  git describe --tags | sed 's/-dev/dev/;s/-rc/rc/;s/[^-]*-g/r&/;s/-/+/g'
}

prepare() {
  cd NetworkManager

  patch -p1 <"$srcdir/0001-fix-ppp-noipv6.patch"
  patch -p1 <"$srcdir/0002-fix-ppp-disable-ipv6.patch"
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

  # NM uses malloc_usable_size in code copied from systemd
  CFLAGS="${CFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"
  CXXFLAGS="${CXXFLAGS/_FORTIFY_SOURCE=3/_FORTIFY_SOURCE=2}"

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
