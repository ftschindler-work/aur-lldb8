# Maintainer: Felix Schindler <aur at felixschindler dot net>
# Contributor: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>

pkgname=lldb8
_pkgname=lldb
pkgver=8.0.1
pkgrel=1
pkgdesc="Next generation, high-performance debugger (8.x)"
arch=('x86_64')
url="https://lldb.llvm.org/"
license=('custom:University of Illinois/NCSA Open Source License')
depends=('llvm8-libs' 'clang8' 'python2' 'python2-six')
makedepends=('llvm8' 'cmake' 'ninja' 'swig')
provides=("$_pkgname=$pkgver")
conflicts=("$_pkgname")
source=(https://github.com/llvm/llvm-project/releases/download/llvmorg-$pkgver/$_pkgname-$pkgver.src.tar.xz
        swig4.patch)
sha256sums=('e8a79baa6d11dd0650ab4a1b479f699dfad82af627cbbcd49fa6f2dc14e131d7'
            'cc5584b8a1f1ea8df23b91915b275aa38ca38ddb6bba4a86d8231210e9cbd06f')
validpgpkeys+=('B6C8F98282B944E3B0D5C2530FC3042E345AD05D') # Hans Wennborg <hans@chromium.org>
validpgpkeys+=('474E22316ABF4785A88C6E8EA2C794A986419D8A') # Tom Stellard <tstellar@redhat.com>

prepare() {
  cd "$srcdir/$_pkgname-$pkgver.src"
  mkdir build

  # https://bugs.llvm.org/show_bug.cgi?id=42284
  patch -Np1 -i ../swig4.patch
}

build() {
  cd "$srcdir/$_pkgname-$pkgver.src/build"

  export CC=clang
  export CXX=clang++

  cmake .. -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DPYTHON_EXECUTABLE=/usr/bin/python2 \
    -DPYTHON_LIBRARY=/usr/lib/libpython2.7.so \
    -DPYTHON_INCLUDE_DIR=/usr/include/python2.7 \
    -DLLVM_LINK_LLVM_DYLIB=ON \
    -DLLDB_USE_SYSTEM_SIX=1
  ninja
}

package() {
  cd "$srcdir/$_pkgname-$pkgver.src/build"

  DESTDIR="$pkgdir" ninja install
  install -Dm644 ../LICENSE.TXT "$pkgdir/usr/share/licenses/$_pkgname/LICENSE"

  # Install possibly outdated man page; better than nothing!
  install -Dm644 ../docs/lldb.1 "$pkgdir/usr/share/man/man1/lldb.1"

  # Remove static libraries
  rm "$pkgdir"/usr/lib/*.a

  # Relocate custom readline.so module which links agaisnt libedit
  mv "$pkgdir"/usr/lib/python2.7/site-packages/{,lldb/}readline.so
  sed -i '2isys.path.insert(1, "/usr/lib/python2.7/site-packages/lldb")' \
    "$pkgdir/usr/lib/python2.7/site-packages/lldb/embedded_interpreter.py"

  # Compile Python scripts
  python2 -m compileall "$pkgdir"
  python2 -O -m compileall "$pkgdir"
}
