Release Process
====================


* * *

###update (commit) version in sources


	fedoracoin-qt.pro
	contrib/verifysfbinaries/verify.sh
	doc/README*
	share/setup.nsi
	src/clientversion.h (change CLIENT_VERSION_IS_RELEASE to true)

###tag version in git

	git tag -s v(new version, e.g. 0.8.0)

###write release notes. git shortlog helps a lot, for example:

	git shortlog --no-merges v(current version, e.g. 0.7.2)..v(new version, e.g. 0.8.0)

* * *

##perform gitian builds

 From a directory containing the fedoracoin source, gitian-builder and gitian.sigs
  
	export SIGNER=(your gitian key, ie bluematt, sipa, etc)
	export VERSION=(new version, e.g. 0.8.0)
	pushd ./fedoracoin
	git checkout v${VERSION}
	popd
	pushd ./gitian-builder

 Fetch and build inputs: (first time, or when dependency versions change)

	mkdir -p inputs; cd inputs/
	wget 'http://miniupnp.free.fr/files/download.php?file=miniupnpc-1.8.tar.gz' -O miniupnpc-1.8.tar.gz
	wget 'https://www.openssl.org/source/openssl-1.0.1e.tar.gz'
	wget 'http://download.oracle.com/berkeley-db/db-4.8.30.NC.tar.gz'
	wget 'http://zlib.net/zlib-1.2.8.tar.gz'
	wget 'ftp://ftp.simplesystems.org/pub/png/src/history/libpng16/libpng-1.6.8.tar.gz'
	wget 'https://fukuchi.org/works/qrencode/qrencode-3.4.3.tar.bz2'
	wget 'https://downloads.sourceforge.net/project/boost/boost/1.55.0/boost_1_55_0.tar.bz2'
	wget 'https://svn.boost.org/trac/boost/raw-attachment/ticket/7262/boost-mingw.patch' -O \ 
	     boost-mingw-gas-cross-compile-2013-03-03.patch
	wget 'https://download.qt-project.org/official_releases/qt/5.2/5.2.0/single/qt-everywhere-opensource-src-5.2.0.tar.gz'
	wget 'https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.bz2'
	cd ..
	./bin/gbuild ../fedoracoin/contrib/gitian-descriptors/boost-linux.yml
	mv build/out/boost-*.zip inputs/
	./bin/gbuild ../fedoracoin/contrib/gitian-descriptors/deps-linux.yml
	mv build/out/fedoracoin-deps-*.zip inputs/
	./bin/gbuild ../fedoracoin/contrib/gitian-descriptors/boost-win.yml
	mv build/out/boost-*.zip inputs/
	./bin/gbuild ../fedoracoin/contrib/gitian-descriptors/deps-win.yml
	mv build/out/fedoracoin-deps-*.zip inputs/
	./bin/gbuild ../fedoracoin/contrib/gitian-descriptors/qt-win.yml
	mv build/out/qt-*.zip inputs/
	./bin/gbuild ../fedoracoin/contrib/gitian-descriptors/protobuf-win.yml
	mv build/out/protobuf-*.zip inputs/

 Build fedoracoind and fedoracoin-qt on Linux32, Linux64, and Win32:
  
	./bin/gbuild --commit fedoracoin=v${VERSION} ../fedoracoin/contrib/gitian-descriptors/gitian-linux.yml
	./bin/gsign --signer $SIGNER --release ${VERSION} --destination ../gitian.sigs/ ../fedoracoin/contrib/gitian-descriptors/gitian-linux.yml
	pushd build/out
	zip -r fedoracoin-${VERSION}-linux-gitian.zip *
	mv fedoracoin-${VERSION}-linux-gitian.zip ../../../
	popd
	./bin/gbuild --commit fedoracoin=v${VERSION} ../fedoracoin/contrib/gitian-descriptors/gitian-win.yml
	./bin/gsign --signer $SIGNER --release ${VERSION}-win --destination ../gitian.sigs/ ../fedoracoin/contrib/gitian-descriptors/gitian-win.yml
	pushd build/out
	zip -r fedoracoin-${VERSION}-win-gitian.zip *
	mv fedoracoin-${VERSION}-win-gitian.zip ../../../
	popd
	popd

  Build output expected:

  1. linux 32-bit and 64-bit binaries + source (fedoracoin-${VERSION}-linux-gitian.zip)
  2. windows 32-bit and 64-bit binaries + installer + source (fedoracoin-${VERSION}-win-gitian.zip)
  3. Gitian signatures (in gitian.sigs/${VERSION}[-win]/(your gitian key)/

repackage gitian builds for release as stand-alone zip/tar/installer exe

**Linux .tar.gz:**

	unzip fedoracoin-${VERSION}-linux-gitian.zip -d fedoracoin-${VERSION}-linux
	tar czvf fedoracoin-${VERSION}-linux.tar.gz fedoracoin-${VERSION}-linux
	rm -rf fedoracoin-${VERSION}-linux

**Windows .zip and setup.exe:**

	unzip fedoracoin-${VERSION}-win-gitian.zip -d fedoracoin-${VERSION}-win
	mv fedoracoin-${VERSION}-win/fedoracoin-*-setup.exe .
	zip -r fedoracoin-${VERSION}-win.zip fedoracoin-${VERSION}-win
	rm -rf fedoracoin-${VERSION}-win

**Perform Mac build:**

  OSX binaries are created by Gavin Andresen on a 64-bit, OSX 10.6 machine.

	./autogen.sh
        SDK=$(xcode-select --print-path)/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.6.sdk
        CXXFLAGS="-mmacosx-version-min=10.6 -isysroot $SDK" ./configure --enable-upnp-default
	make
	export QTDIR=/opt/local/share/qt4  # needed to find translations/qt_*.qm files
	T=$(contrib/qt_translations.py $QTDIR/translations src/qt/locale)
        export CODESIGNARGS='--keychain ...path_to_keychain --sign "Developer ID Application: BITCOIN FOUNDATION, INC., THE"'
	python2.7 contrib/macdeploy/macdeployqtplus Fedoracoin-Qt.app -sign -add-qt-tr $T -dmg -fancy contrib/macdeploy/fancy.plist

 Build output expected: Fedoracoin-Qt.dmg

###Next steps:

* Code-sign Windows -setup.exe (in a Windows virtual machine using signtool)
 Note: only Gavin has the code-signing keys currently.

* upload builds to SourceForge

* create SHA256SUMS for builds, and PGP-sign it

* update fedoracoin.org version
  make sure all OS download links go to the right versions
  
* update download sizes on fedoracoin.org/_templates/download.html

* update forum version

* update wiki download links

* update wiki changelog: [https://en.fedoracoin.it/wiki/Changelog](https://en.fedoracoin.it/wiki/Changelog)

Commit your signature to gitian.sigs:

	pushd gitian.sigs
	git add ${VERSION}/${SIGNER}
	git add ${VERSION}-win/${SIGNER}
	git commit -a
	git push  # Assuming you can push to the gitian.sigs tree
	popd

-------------------------------------------------------------------------

### After 3 or more people have gitian-built, repackage gitian-signed zips:

From a directory containing fedoracoin source, gitian.sigs and gitian zips

	export VERSION=(new version, e.g. 0.8.0)
	mkdir fedoracoin-${VERSION}-linux-gitian
	pushd fedoracoin-${VERSION}-linux-gitian
	unzip ../fedoracoin-${VERSION}-linux-gitian.zip
	mkdir gitian
	cp ../fedoracoin/contrib/gitian-downloader/*.pgp ./gitian/
	for signer in $(ls ../gitian.sigs/${VERSION}/); do
	 cp ../gitian.sigs/${VERSION}/${signer}/fedoracoin-build.assert ./gitian/${signer}-build.assert
	 cp ../gitian.sigs/${VERSION}/${signer}/fedoracoin-build.assert.sig ./gitian/${signer}-build.assert.sig
	done
	zip -r fedoracoin-${VERSION}-linux-gitian.zip *
	cp fedoracoin-${VERSION}-linux-gitian.zip ../
	popd
	mkdir fedoracoin-${VERSION}-win-gitian
	pushd fedoracoin-${VERSION}-win-gitian
	unzip ../fedoracoin-${VERSION}-win-gitian.zip
	mkdir gitian
	cp ../fedoracoin/contrib/gitian-downloader/*.pgp ./gitian/
	for signer in $(ls ../gitian.sigs/${VERSION}-win/); do
	 cp ../gitian.sigs/${VERSION}-win/${signer}/fedoracoin-build.assert ./gitian/${signer}-build.assert
	 cp ../gitian.sigs/${VERSION}-win/${signer}/fedoracoin-build.assert.sig ./gitian/${signer}-build.assert.sig
	done
	zip -r fedoracoin-${VERSION}-win-gitian.zip *
	cp fedoracoin-${VERSION}-win-gitian.zip ../
	popd

- Upload gitian zips to SourceForge

- Announce the release:

  - Add the release to fedoracoin.org: https://github.com/fedoracoin/fedoracoin.org/tree/master/_releases

  - Release sticky on fedoracointalk: https://fedoracointalk.org/index.php?board=1.0

  - Fedoracoin-development mailing list

  - Optionally reddit /r/Fedoracoin, ...

- Celebrate 
