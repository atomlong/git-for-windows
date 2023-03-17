# Maintainer: Alexey Pavlov <alexpux@gmail.com>
# Maintainer: Ray Donnelly <mingw.android@gmail.com>

_pkgname=git
pkgname=${_pkgname}-for-windows
pkgver=2.40.0
pkgrel=1
pkgver_win=${pkgver}.windows.${pkgrel}
pkgdesc="The fast distributed version control system"
arch=('i686' 'x86_64')
url="https://git-scm.com/"
license=('GPL2')
groups=('VCS')
depends=('curl'
         #'info'
         'libpcre2_8'
         'libexpat'
         'libintl'
         'nano'
         'openssh'
         'openssl'
         'perl-Error'
         'perl>=5.14.0'
         'perl-Authen-SASL'
         'perl-libwww'
         'perl-MIME-tools'
         'perl-Net-SMTP-SSL'
         'perl-TermReadKey')
makedepends=('asciidoc'
             'autotools'
             'docbook-xml'
             'docbook-xsl'
             #'docbook2x'
             'gcc'
             'less'
             'libexpat-devel'
             'libcurl-devel'
             'libiconv-devel'
             'gettext-devel'
             'pcre2-devel'
             'python'
             'rsync'
             #'texinfo'
             'xmlto')
optdepends=(#'tk: gitk and git gui'
            'python: various helper scripts'
            'subversion: git svn'
            #'cvsps: git cvsimport'
            )
replaces=('git-core')
provides=('git-core')
#options=('debug' '!strip')
source=("${pkgname}-${pkgver_win}.tar.gz"::https://github.com/git-for-windows/git/archive/v${pkgver_win}.tar.gz
        1.7.9-cygwin.patch
        git-1.8.4-msys2.patch
        git-2.8.2-Cygwin-Allow-DOS-paths.patch
        git-tcsh-completion-fixes.patch
        git-2.35.1-git-gui.patch
		git-credential-wincred-fixes.patch)
sha256sums=('53e207293bc226ad129ed0ed6ea8d314c01a060dcfff5774f54c4639fde5f427'
            '32baa705d76789d82316a1325e61c9a261114eaa9814dca9c05683bf63069dea'
            '9bc4da5022c5004c4c5b45417b25c6636ddf0ac338574a624c2c775d4394546d'
            '369c52227d10803dd2ace37053cbcbffe63fad23b313bfbe3dce0f3ecd31ca4e'
            '6f25aae9e92469d5e284dcf372e15ee0e57ff91531e691e7391f9bfb78f73626'
            '989473b9ea683fcea7761d3f82a37653c63c2df7be795e4edcdeffaa0c22c5b6'
			'3abeca1dc20c29644827f5701dbd05ef4f236b69907970ad51863a312307bed2')

# Helper macros to help make tasks easier #
apply_patch_with_msg() {
  for _fname in "$@"
  do
    msg2 "Applying ${_fname}"
    patch -Nbp1 -i "${srcdir}"/${_fname}
  done
}

prepare() {
  cd "${srcdir}/${_pkgname}-${pkgver_win}"

  rm -f compat/win32/git.manifest compat/win32/resource.rc
  apply_patch_with_msg \
    1.7.9-cygwin.patch \
    git-1.8.4-msys2.patch \
    git-2.8.2-Cygwin-Allow-DOS-paths.patch \
    git-tcsh-completion-fixes.patch \
	git-credential-wincred-fixes.patch

  # https://github.com/gitgitgadget/git/pull/612
  # hacky variant, since we have no cygwin tcl
  apply_patch_with_msg \
    git-2.35.1-git-gui.patch

  make configure
}

build() {
  export PYTHON_PATH='/usr/bin/python'
  cd "${srcdir}/${_pkgname}-${pkgver_win}"

  local CYGWIN_CHOST="${CHOST/-msys/-cygwin}"
  ./configure \
    --build=${CYGWIN_CHOST} \
    --prefix=/usr \
    --sysconfdir=/etc \
    --libexecdir=/usr/lib \
    --with-libpcre2 \
    --with-editor=nano \
    --htmldir=/usr/share/doc/git/html \
    --mandir=/usr/share/man

  make INSTALLDIRS=vendor all
  make -j1 man
  #make -j1 info
  make -C contrib/subtree prefix=/usr all
}

check() {
  export PYTHON_PATH='/usr/bin/python'
  cd "${srcdir}/${_pkgname}-${pkgver_win}"
  local jobs
  jobs=$(expr "$MAKEFLAGS" : '.*\(-j[0-9]*\).*')
  mkdir -p /tmp/git-test
  # We used to use this, but silly git regressions:
  #GIT_TEST_OPTS="--root=/dev/shm/" \
  # https://comments.gmane.org/gmane.comp.version-control.git/202020
  make \
    NO_SVN_TESTS=y \
    DEFAULT_TEST_TARGET=prove \
    GIT_PROVE_OPTS="$jobs -Q" \
    GIT_TEST_OPTS="--root=/tmp/git-test" \
    test
}

package() {
  export PYTHON_PATH='/usr/bin/python'
  cd "${srcdir}/${_pkgname}-${pkgver_win}"
  make INSTALLDIRS=vendor DESTDIR="$pkgdir" install
  make INSTALLDIRS=vendor DESTDIR="${pkgdir}" install-man
  #make INSTALLDIRS=vendor DESTDIR="${pkgdir}" install-info

  # bash completion
  mkdir -p "${pkgdir}"/usr/share/bash-completion/completions/
  install -m644 ./contrib/completion/git-completion.bash "${pkgdir}"/usr/share/bash-completion/completions/git
  # fancy git prompt
  mkdir -p "${pkgdir}"/usr/share/git/
  install -m644 ./contrib/completion/git-prompt.sh "${pkgdir}"/usr/share/git/git-prompt.sh
  # subtree installation
  sed "s|libexec/git-core|lib/git-core|" -i contrib/subtree/Makefile
  make -C contrib/subtree prefix=/usr DESTDIR="${pkgdir}" install

  # the rest of the contrib stuff
  cp -a ./contrib/* ${pkgdir}/usr/share/git/

  # scripts are for python
  sed -i 's|#![ ]*/usr/bin/env python$|#!/usr/bin/python|' \
    $(find "${pkgdir}" -name '*.py') \
    "${pkgdir}"/usr/lib/git-core/git-p4 \
    "${pkgdir}"/usr/share/git/remote-helpers/git-remote-bzr \
    "${pkgdir}"/usr/share/git/remote-helpers/git-remote-hg

  # Remove *.orig files
  find "${pkgdir}/usr" -type f -name "*.orig" -exec rm -f {} \;
}
