# Maintainer: Tritron <jacek@hebe.u>
# Contributor krprescott
# Contributor Mevordel
# This pkgbuild is a modified xen4 pkgbuild. Most credits go to that maintainer.

pkgname=xen-4.3-git
pkgver=20130730
pkgrel=1
pkgdesc="Xen 4.3 stable  (hypervisor tools and doc) git"
arch=(i686 x86_64)
url="http://xen.org/"
license="GPL"
depends=('xz' 'nss' 'libpng' 'bzip2' 'iproute' 'libgl'  'bridge-utils' 'python2' 'sdl' 'zlib' 'urlgrabber' 'e2fsprogs' 'bin86' 'libaio' 'pkgconfig' 'gnutls' 'lzo2' 'libseccomp' 'vde2' 'glibc' 'yajl' 'inetutils' 'python' 'iasl' 'spice' 'spice-protocol' 'usleep' 'libiscsi' 'bluez-libs' 'libcacard' 'usbredir' )
makedepends=('dev86' 'texi2html' 'git' 'libgl' 'ghostscript' 'markdown' 'ocaml-findlib' 'spice' 'libiscsi' 'libcacard' 'libaio' 'usbredir' 'spice-protocol' 'vde2' 'libseccomp')
conflicts=('xen' 'xen3' 'xen4' 'xen-hv-tools' 'libxen4')
provides=('xen')
options=(!strip !buildflags)
source=(archlinux.patch
        qemu.patch
        seabios.patch
        oxen.patch
        xen.conf
	rtc.patch
        rc.status
        texi2html.patch
        lrt.patch
        proc-xen.mount
        xenconsoled.service
        xendomains.service
        xendomU.service
        xenstored.service
        var-lib-xenstored.mount        
        $pkgname.conf        
        09_xen
        hotfix.patch 
        bios_workaround.patch
        bios_vtd_workaround.patch)

md5sums=('2ead4dc56db3c93fb087af6f55d49509'
         '08e40f0c74fb2e68b67778fff96a4214'
         'ff8489556b32bcc76c62f20ada3c6d38'
         '486b53458e0833b9796b282d487aebf5'
         '6eadafc3b517bcb5a6307370d0ffda3b'
         '8eefbddc84fc9062736b20662519c2f1'
         '2c948bb3d7d3bd8ea824cc01fdb9889b'
         'c2015119cb07230917ce463753c4f1fc'
         '6c193a17fa283a05c12d0d4a42f5954d'
         '3252fa21362fd55246f9d8b923070151'
         '68ab1e5137eaee06cd8be3d546043566'
         'a574b64b15c69c8b4927a91d3e8a03e8'
         '2c493e71f780ed2bcc69a14bef47f755'
         'd2af0cebec989b2df1ae10ae26bb6c2f'
         'f490d4087fb3d4866c0363d0d013f054'
         '1043c13f40f82ca995f569ffcaa2dd7d'
         '1eb1de5675e4499018a37c3a5de973fe'
         '08dab5849f90d892e0b0d82556b3cdfc'
         '7bd02d09f3f528bf63294d4a9ac19dd6'
         '5d5c4d040325dbe222d8f24efdbf0dc4')

_gitroot="git://xenbits.xen.org/xen.git"
_gitname="xen"
	
pkgver() {
  date +%Y%m%d
}
	
prepare() {
  cd "$srcdir"
  if [ -d $srcdir/$_gitname ] ; then
    msg "Git checkout:  Updating existing tree"
    cd $_gitname && git pull origin
    msg "Git checkout:  Tree has been updated"
  else
    msg "Git checkout:  Retrieving sources"
    git clone -b stable-4.3 $_gitroot
  fi
  rm -rf "$srcdir/$_gitname-build"
  cp -r "$srcdir/$_gitname" "$srcdir/$_gitname-build"
  cd "$srcdir/$_gitname-build"
  patch -Np1 -F99 -i "$srcdir/archlinux.patch"
  patch -Np1 -F99 -i "$srcdir/qemu.patch"
  patch -Np1 -F99 -i "$srcdir/seabios.patch"
  patch -Np1 -F99 -i "$srcdir/hotfix.patch"
  patch -Np1 -F99 -i "$srcdir/oxen.patch"
  patch -Np1 -F99 -i "$srcdir/rtc.patch"
  patch -Np1 -i "$srcdir/texi2html.patch"
  patch -Np1 -i "$srcdir/lrt.patch"
  patch -Np1 -i "$srcdir/bios_workaround.patch"
  patch -Np1 -i "$srcdir/bios_vtd_workaround.patch"
  sed -i 's:/sbin:/bin:' config/StdGNU.mk
}


build()  {
  cd "$srcdir/$_gitname-build"
   export COMPILER_PATH=/usr/bin
   unset CFLAGS LDFLAGS
  ./autogen.sh 
  PYTHON=/usr/bin/python2 ./configure --prefix=/usr --localstatedir=/run --enable-spice --enable-usb-redir --enable-stubdom  --enable-debug  --enable-xen-pci-passthrough --enable-bluez --enable-libiscsi --enable-vhost-net --enable-linux-aio --enable-vde --enable-nptl --enable-libiscsi --enable-smartcard-nss --enable-targets=X86_64-pep
}
package() {
  mkdir -p $pkgdir/boot/efi/EFI/arch
  cd "$srcdir/$_gitname-build"
  make LANG=C PYTHON=python2 DESTDIR="$pkgdir" install-xen install-tools
  make LANG=C -j1 PYTHON=python2 DESTDIR="$pkgdir" install-stubdom
  make LANG=C PYTHON=python2  DESTDIR=$pkgdir  install-stubdom
  mkdir -p $pkgdir/boot/efi/EFI/arch
  cd ../
  for f in ${source[@]}; do
        [[ $f =~ .mount || $f =~ .service ]] && install -Dm644 $f "$pkgdir"/usr/lib/systemd/system/$f
    done
    install -Dm644 $pkgname.conf "$pkgdir"/etc/modules-load.d/$pkgname.conf
    install -Dm755 09_xen "$pkgdir"/etc/grub.d/09_xen
	   cd "$pkgdir"
    if [[ -d usr/lib64 ]]; then
        cd usr/
        cp -r lib64/* lib/
        rm -rf lib64
    fi
  cd $pkgdir
  mv etc/{init,rc}.d
  mv usr/etc/qemu/ etc/
  rm -rf usr/local/share/
  mv etc/rc.d/xendomains etc/xen/scripts/xendomains  
  mkdir -p etc/conf.d
  mv etc/default/xendomains etc/conf.d/xendomains
  ############ kill unwanted stuff ############
  # stubdom: newlib
  rm -rf $pkgdir/usr/*-xen-elf

# hypervisor symlinks
rm -rf $pkgdir/boot/xen-4.3.gz
rm -rf $pkgdir/boot/xen-4.gz
rm -rf $pkgdir/boot/xen.gz

# adhere to Static Library Packaging Guidelines
rm -rf $pkgdir/usr/lib/*.a 	
# Fix errors from deprecated xend
rm etc/udev/rules.d/xend.rules    

mkdir -p usr/lib/tmpfiles.d/
cp -a ../../xen.conf usr/lib/tmpfiles.d/
cp -a ../../rc.status   etc 
}
