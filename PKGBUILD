# Maintainer: Daniel Micay <danielmicay@gmail.com>
pkgname=cling-svn
pkgver=48300
pkgrel=1
pkgdesc="Interactive C++ interpreter and JIT compiler"
arch=('i686' 'x86_64')
url="http://root.cern.ch/drupal/content/cling"
#license=('custom:University of Illinois/NCSA Open Source License')
license=(custom)
depends=('gcc-libs' 'libffi' 'python2' "gcc")
makedepends=('svn')
provides=('clang' 'llvm')
conflicts=(llvm llvm-svn llvm-ocaml clang clang-analyzer)
options=(!buildflags)
source=(svn+http://root.cern.ch/svn/root/trunk/interpreter/cling enable-lto.patch)
sha256sums=(SKIP '05f77e1e92ba79ac5e93aa68fd850fc3b1eac970bc5b8e46de91378cdbf74c8f')

_llvmrepo=http://llvm.org/svn/llvm-project

build() {
  # use python2 for the build
  mkdir -p "$srcdir/python2-path"
  ln -sf /usr/bin/python2 "$srcdir/python2-path/python"
  export PATH="$srcdir/python2-path:$PATH"

  local rev=$(cat cling/LastKnownGoodLLVMSVNRevision.txt)

  # llvm
  if [[ -d llvm/.svn ]]; then
    (cd llvm && svn up -r "$rev")
  else
    svn co $_llvmrepo/llvm/trunk --config-dir ./ -r "$rev" llvm
  fi

  # clang
  if [[ -d clang/.svn ]]; then
    (cd clang && svn up -r "$rev")
  else
    svn co $_llvmrepo/cfe/trunk --config-dir ./ -r "$rev" clang
  fi

  # compiler-rt
  if [[ -d compiler-rt/.svn ]]; then
    (cd compiler-rt && svn up -r "$rev")
  else
    svn co $_llvmrepo/compiler-rt/trunk --config-dir ./ -r "$rev" compiler-rt
  fi

  [[ -d llvm-build ]] && rm -r llvm-build

  cp -a llvm llvm-build
  cp -a clang llvm-build/tools/clang
  cp -a compiler-rt llvm-build/projects/compiler-rt
  cp -a cling llvm-build/tools/cling

  cd llvm-build

  # Use gold instead of default linker to make -flto work
  patch -d tools/clang -Np0 -i "$srcdir/enable-lto.patch"

  cat tools/cling/patches/*.diff | patch -p0
  ./configure \
    --prefix=/usr \
    --enable-optimized \
    --disable-assertions \
    --enable-targets=host \
    --with-binutils-include=/usr/include
  make
}

package() {
  cd llvm-build
  make DESTDIR="$pkgdir" install
}
