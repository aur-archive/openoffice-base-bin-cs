
# Maintainer: Kurt J. Bosch <kjb-temp-2009 at alpenjodel.de>
# Modified for Czech language by Jirka Zelinka j.zelinka@centrum.cz

_lang=cs
pkgname=openoffice-base-bin-cs
pkgver=3.3.0
pkgrel=2
pkgdesc="OpenOffice.org - Repackaged from upstream"
arch=('i686' 'x86_64')
url="http://openoffice.org/"
license=('LGPL3') # more below
# Some depends can be found in the README, some with namcap, some with _extract_depends() below
depends=(
	'desktop-file-utils'        # install script
	'freetype2'
	'glibc>=2.5'
	'gtk2>=2.10.4'
	'hicolor-icon-theme'
	'sh'                        # freedesktop-menus
	'shared-mime-info'          # install script
)
makedepends=(
	'coreutils' 'findutils' 'rpmextract'
#	'binutils' 'tar' # _debunpack()
)
optdepends=(
	'libgail-gnome: GNOME Assistive Technology'
	'mime-types: provides /etc/mime.types'
	'openoffice-de-bin: Language pack (example)'
)
backup=() # more below
options=(!strip docs)
install=openoffice-bin-cs.install

_url="http://download.services.openoffice.org/files"
_arch="-ARCH-" # feed the AUR ;-)
_src_extra=""
case "$CARCH"
in "i686"   ) _arch="x86"   ;                     md5sums=( 'f3f4984212021253df3507767eccc5b0' )
;; "x86_64" ) _arch="x86-64"; _src_extra="-wJRE"; md5sums=( '00c9d69a798d5d7437a30a93f6f4df97' )
esac
_dirs=( "${srcdir}"/OOO330_m20_native_packed-1_${_lang}.9567 )

# deb tarballs contain no freedesktop integration !
source=("${_url}"/localized/cs/${pkgver}/OOo_${pkgver}_Linux_${_arch}_install-rpm${_src_extra}_cs.tar.gz )

#_debunpack() { ar -p "$1" data.tar.gz | tar -xz; }

package() {
	cd "${pkgdir}"
	# unpack RPMs and DEBs
	local dir file
	for dir in "${_dirs[@]}"; do
		for file in $( cd "${dir}" && find -type f \( -name '*.rpm' -o -name '*.deb' \) ); do
			if [[ $file == */desktop-integration/* && $file != *-freedesktop-menus-* ]] ||
			   [[ $file == */jre* ]]; then
				msg2 "Skipping ${file#./}"
				continue
			fi
			msg2 "Extracting ${file#./}"
			case $file
			in *.rpm ) rpmextract.sh "${dir}/${file}"
			;; *.deb ) _debunpack    "${dir}/${file}"
			esac
		done
	done
	local base=opt
	# link license files
	local file i=0
	for file in $( find "${base}" -type f -name 'THIRDPARTY*LICENSE*.html' ); do
		mkdir -p usr/share/licenses/${pkgname}
		ln -sT /${file} usr/share/licenses/${pkgname}/THIRDPARTY-$((++i)).html
		license+=( "custom:THIRDPARTY-$i" )
	done
	# add backups
	backup+=( ${base}/openoffice.org3/program/sofficerc )
	# link mozilla plugin
	mkdir -p usr/lib/mozilla/plugins
	ln -s -t usr/lib/mozilla/plugins /${base}/openoffice.org3/program/libnpsoplugin.so
}

# Function for listing external dependencies
_extract_depends() {
	(( $# == 1 )) || { echo "Usage: $FUNCNAME <RPMS-directory-path>" >&2; return 1; }
	(
		cd "$1"
		for f in $( find -type f -name '*.rpm' ); do
			r=$( rpmmeta -t requirename $f | sed -re 's;(ooobasis|openoffice|rpmlib)[^ ]*;;g' )
			[[ $r ]] && echo ${f#./} $r
		done
	)
}

# Function for extracting install script code
_extract_install() {
	(( $# == 1 )) || { echo "Usage: $FUNCNAME <RPMS-directory-path>" >&2; return 1; }
	cat <<EOF
## arg 1:  the new package version
post_install() {

  ## code from freedesktop-menus rpm tag postin

  #### Inappropriate parts should be removed:
  #### - mime.type stuff is already provided by mime-types package
  #### - /etc/mailcap does not exist on ArchLinux normaly
  #### - Don't use 'which' because tools are already in depends

EOF
	rpmmeta -t postin "${1}"/desktop-integration/openoffice*-freedesktop-menus-*.noarch.rpm
	cat <<EOF
}

## arg 1:  the new package version
## arg 2:  the old package version
post_upgrade() {
  post_install \$1
}

## arg 1:  the old package version
post_remove() {

  ## code from freedesktop-menus rpm tag postun

  #### inappropriate parts should be removed

EOF
	rpmmeta -t postun "${1}"/desktop-integration/openoffice*-freedesktop-menus-*.noarch.rpm
	cat <<EOF
}
EOF
}
