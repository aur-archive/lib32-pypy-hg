# Original author: Sven-Hendrik Haase (svenstaro) <sh@lutzhaase.com>
# Maintainer: Zack Buhman (buhman) <zack@buhman.org>
# Contributor: William Giokas (KaiSforza) <1007380@gmail.com>

USE_PYPY=0
USE_SHARED=1

pkgname=lib32-pypy-hg
pkgver=74336.04f76bbfd718
pkgrel=1
pkgdesc="A Python implementation written in Python, JIT enabled"
url="http://codespeak.net/pypy/"
arch=('x86_64')
depends=('lib32-libffi' 'lib32-openssl' 'lib32-expat' 'lib32-ncurses'
         'lib32-zlib' 'lib32-bzip2')
options=('debug' 'strip' '!emptydirs')
# Force cpython2 to be installed for mercurial
makedepends=('python2>=2.7' 'mercurial' 'lib32-gdbm' 'lib32-sqlite'
             'lib32-tcl' 'lib32-tk')
if ((USE_PYPY)); then
    makedepends+=('lib32-pypy')
else
    makedepends+=('lib32-python2')
fi
conflicts=('lib32-pypy')
provides=('lib32-pypy=2.5')
optdepends=('lib32-sqlite: sqlite3 module'
            'lib32-gdbm: gdbm module'
            'lib32-tk: lib-tk modules'
            'lib32-tcl: lib-tk modules')
license=('custom:MIT')

source=('hg+https://bitbucket.org/pypy/pypy'
        '0001-fix-compiler.patch')
md5sums=('SKIP'
         'af3f05d040c4b575c380b0b33de842a9')

_check_python() {
  _PYTHON=python2-32
  if ((USE_PYPY)) && which pypy-32 &> /dev/null; then
      _PYTHON=pypy-32
  fi
}

pkgver() {
  cd "${srcdir}"/pypy
  echo "$(hg id -n).$(hg id -i)"
}

prepare() {
  cd pypy

  patch -Np1 < ../0001-fix-compiler.patch
}

build() {
  cd "${srcdir}/pypy/pypy/goal"

  _check_python

  translate_options=()

  if ((USE_SHARED)); then
      translate_options+=('--shared')
  fi

  CC='gcc -m32'
  CXX='g++ -m32'
  CFLAGS+=' -march=i686'
  LDFLAGS+=' -L/usr/lib32'

  export CC
  export CXX
  export CFLAGS
  export LDFLAGS

  msg2 'Start Translation'
  $_PYTHON ../../rpython/bin/rpython -Ojit "${translate_options[@]}" \
           --output=pypy-c-32 targetpypystandalone
}

package() {
  cd "${srcdir}"/pypy/pypy/tool/release

  _check_python

  CC='gcc -m32'
  CXX='g++ -m32'
  CFLAGS+=' -march=i686'
  LDFLAGS+=' -L/usr/lib32'

  export CC
  export CXX
  export CFLAGS
  export LDFLAGS

  LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:../../goal/" $_PYTHON package.py \
                 --override_pypy_c ../../goal/pypy-c-32 ../../../ \
                 pypy-32 pypy-c-32 "${srcdir}"/${pkgname}.tar.bz2

  mkdir -p "${pkgdir}"/opt
  tar x -C "${pkgdir}"/opt -f "${srcdir}"/${pkgname}.tar.bz2

  if ((USE_SHARED)); then
      mkdir -p "${pkgdir}"/opt/pypy-32/lib
      cp ../../goal/libpypy-c-32.so "${pkgdir}"/opt/pypy-32/lib/
      mkdir -p "${pkgdir}"/usr/lib32
      ln -s /opt/pypy-32/lib/libpypy-c-32.so \
         "${pkgdir}"/usr/lib32/libpypy-c-32.so
  fi

  mkdir -p "${pkgdir}"/usr/bin
  ln -s /opt/pypy-32/bin/pypy-c-32 "${pkgdir}"/usr/bin/pypy-32

  install -Dm644 "${pkgdir}"/opt/pypy-32/LICENSE \
          "${pkgdir}"/usr/share/licenses/pypy-32/LICENSE
}
