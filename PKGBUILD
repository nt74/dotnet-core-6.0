# Maintainer: Nikos Toutountzoglou <nikos.toutou@gmail.com>

pkgname=dotnet-core-6.0
pkgver=6.0.16
pkgrel=1
pkgdesc="Installation of dotnet-core-6.0 SDK/Runtime via the official installation script"
arch=(any)
url="https://www.microsoft.com/net/core"
license=(MIT)
provides=("dotnet-core-6.0=${pkgver}")
conflicts=(
	dotnet-core-6.0
	dotnet-runtime-6.0
	dotnet-sdk-6.0
	dotnet-targeting-pack-6.0
	dotnet-core-6.0-bin
	dotnet-targeting-pack-6.0-bin
	dotnet-sdk-6.0-bin
)
depends=()
makedepends=()
source=(dotnet-install.sh::"https://dot.net/v1/dotnet-install.sh")
b2sums=('5380e01cccf119769ba711fde97cb5f2b73f3a24b9c00144de5e49e1e4c08c34e210d4c302fb50ad3f0d3d05a24e4b98c16636755a5b69838731a55c3266f8ff')

prepare() {
	# Check if /tmp directory exists
	if [ ! -d "/tmp" ]; then
		echo "Error: Couldn't find '/tmp' directory, exiting."
		exit 1
	fi
	
	cd "${srcdir}"
	chmod +x dotnet-install.sh
}

build() {
	# Set correct CPU architecture
	if [ $CARCH = 'x86' ]; then msarch=x86;
	elif [ $CARCH = 'x86_64' ]; then msarch=x64;
	elif [ $CARCH = 'armv7h' ]; then msarch=arm;
	elif [ $CARCH = 'aarch64' ]; then msarch=arm64;
	else msarch=auto; fi

	cd "${srcdir}"

	# DISABLE tmpfs for ARM devices
	# Workaround for 'out of space' tmpfs (/tmp) directory
	if [ $CARCH = 'armv7h' ] && [ $(systemctl is-active tmp.mount) = 'active' ]; then sudo systemctl stop tmp.mount;
	elif [ $CARCH = 'aarch64' ] && [ $(systemctl is-active tmp.mount) = 'active' ]; then sudo systemctl stop tmp.mount; fi

	# Download dotnet-core-6.0 files
	# Requires /tmp directory to unpack
	./dotnet-install.sh \
	--channel 6.0 \
	--architecture ${msarch} \
	--install-dir dotnet \
	--no-path \
	--verbose

	# ENABLE tmpfs for ARM devices
	# Workaround for 'out of space' tmpfs (/tmp) directory
	if [ $CARCH = 'armv7h' ] && [ $(systemctl is-active tmp.mount) = 'inactive' ]; then sudo systemctl start tmp.mount;
	elif [ $CARCH = 'aarch64' ] && [ $(systemctl is-active tmp.mount) = 'inactive' ]; then sudo systemctl start tmp.mount; fi	
}

package() {
	cd "${srcdir}/dotnet"
	install -dm 755 "${pkgdir}"/usr/{bin,lib}
	install -dm 755 "${pkgdir}"/usr/share/{dotnet,dotnet/shared,dotnet/packs,licenses,licenses/"${pkgname}"}
	cp -dr --no-preserve='ownership' dotnet "${pkgdir}"/usr/share/dotnet/dotnet
	cp -dr --no-preserve='ownership' host shared packs sdk sdk-manifests templates "${pkgdir}"/usr/share/dotnet/
	cp -dr --no-preserve='ownership' "LICENSE.txt" "ThirdPartyNotices.txt" "${pkgdir}"/usr/share/licenses/"${pkgname}"
	ln -s /usr/share/dotnet/dotnet "${pkgdir}"/usr/bin/dotnet
	ln -s /usr/share/dotnet/host/fxr/"${pkgver}"/libhostfxr.so "${pkgdir}"/usr/lib/libhostfxr.so
}
