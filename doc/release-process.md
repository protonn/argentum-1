Release Process
====================

* update translations (ping wumpus, Diapolo or tcatm on IRC)
* see https://github.com/bitcoin/bitcoin/blob/master/doc/translation_process.md#syncing-with-transifex

* * *

###update (commit) version in sources

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

 From a directory containing the argentum source, gitian-builder and gitian.sigs
  
	export SIGNER=(your gitian key, ie bluematt, sipa, etc)
	export VERSION=(new version, e.g. 0.8.0)
	pushd ./argentum
	git checkout v${VERSION}
	popd
	pushd ./gitian-builder

 Fetch and build inputs: (first time, or when dependency versions change)

	mkdir -p inputs; cd inputs/
	wget 'http://miniupnp.free.fr/files/download.php?file=miniupnpc-1.9.20151008.tar.gz' -O miniupnpc-1.9.20151008.tar.gz
	wget 'https://www.openssl.org/source/openssl-1.0.1l.tar.gz'
	wget 'http://download.oracle.com/berkeley-db/db-5.1.29.NC.tar.gz'
	wget 'http://zlib.net/zlib-1.2.8.tar.gz'
	wget 'https://downloads.sourceforge.net/project/libpng/libpng16/older-releases/1.6.8/libpng-1.6.8.tar.gz'
	wget 'https://fukuchi.org/works/qrencode/qrencode-3.4.3.tar.bz2'
	wget 'https://downloads.sourceforge.net/project/boost/boost/1.55.0/boost_1_55_0.tar.bz2'
	wget 'https://svn.boost.org/trac/boost/raw-attachment/ticket/7262/boost-mingw.patch' -O \
	     boost-mingw-gas-cross-compile-2013-03-03.patch
	wget 'https://download.qt-project.org/official_releases/qt/5.2/5.2.0/single/qt-everywhere-opensource-src-5.2.0.tar.gz'
	wget 'https://download.qt-project.org/archive/qt/4.6/qt-everywhere-opensource-src-4.6.4.tar.gz'
	wget 'https://protobuf.googlecode.com/files/protobuf-2.5.0.tar.bz2'
	cd ..
	./bin/gbuild ../argentum/contrib/gitian-descriptors/boost-linux.yml
	mv build/out/boost-*.zip inputs/
	./bin/gbuild ../argentum/contrib/gitian-descriptors/deps-linux.yml
	mv build/out/argentum-deps-*.zip inputs/
	./bin/gbuild ../argentum/contrib/gitian-descriptors/qt-linux.yml
	mv build/out/qt-*.tar.gz inputs/
	./bin/gbuild ../argentum/contrib/gitian-descriptors/boost-win.yml
	mv build/out/boost-*.zip inputs/
	./bin/gbuild ../argentum/contrib/gitian-descriptors/deps-win.yml
	mv build/out/argentum-deps-*.zip inputs/
	./bin/gbuild ../argentum/contrib/gitian-descriptors/qt-win.yml
	mv build/out/qt-*.zip inputs/
	./bin/gbuild ../argentum/contrib/gitian-descriptors/protobuf-win.yml
	mv build/out/protobuf-*.zip inputs/

 The expected SHA256 hashes of the intermediate inputs are:

    0331de2b05c275986545e8d2d6a4becf625e5935034803dc508c9d641e70c7cb  argentum-deps-linux32-gitian-r10.zip
    d99caabf13011494dd147b4ffad1626480889f5080270a1a04e1a7adc543cc4f  argentum-deps-linux64-gitian-r10.zip
    f29b7d9577417333fb56e023c2977f5726a7c297f320b175a4108cf7cd4c2d29  boost-linux32-1.55.0-gitian-r1.zip
    88232451c4104f7eb16e469ac6474fd1231bd485687253f7b2bdf46c0781d535  boost-linux64-1.55.0-gitian-r1.zip
    57e57dbdadc818cd270e7e00500a5e1085b3bcbdef69a885f0fb7573a8d987e1  qt-linux32-4.6.4-gitian-r1.tar.gz
    60eb4b9c5779580b7d66529efa5b2836ba1a70edde2a0f3f696d647906a826be  qt-linux64-4.6.4-gitian-r1.tar.gz
    60dc2d3b61e9c7d5dbe2f90d5955772ad748a47918ff2d8b74e8db9b1b91c909  boost-win32-1.55.0-gitian-r6.zip
    f65fcaf346bc7b73bc8db3a8614f4f6bee2f61fcbe495e9881133a7c2612a167  boost-win64-1.55.0-gitian-r6.zip
    3783d98ac49256f11381d4eadffed5b51c1779afedca43ffef6cf8998d1db9c5  argentum-deps-win32-gitian-r17.zip
    b579a9af8a8b77d542738cf93e5a12bfb0957dade0281bebd96a1619fc587855  argentum-deps-win64-gitian-r17.zip
    963e3e5e85879010a91143c90a711a5d1d5aba992e38672cdf7b54e42c56b2f1  qt-win32-5.2.0-gitian-r3.zip
    751c579830d173ef3e6f194e83d18b92ebef6df03289db13ab77a52b6bc86ef0  qt-win64-5.2.0-gitian-r3.zip
    e2e403e1a08869c7eed4d4293bce13d51ec6a63592918b90ae215a0eceb44cb4  protobuf-win32-2.5.0-gitian-r4.zip
    a0999037e8b0ef9ade13efd88fee261ba401f5ca910068b7e0cd3262ba667db0  protobuf-win64-2.5.0-gitian-r4.zip

 Build argentumd and argentum-qt on Linux32, Linux64, and Win32:
  
	./bin/gbuild --commit argentum=v${VERSION} ../argentum/contrib/gitian-descriptors/gitian-linux.yml
	./bin/gsign --signer $SIGNER --release ${VERSION} --destination ../gitian.sigs/ ../argentum/contrib/gitian-descriptors/gitian-linux.yml
	pushd build/out
	zip -r argentum-${VERSION}-linux-gitian.zip *
	mv argentum-${VERSION}-linux-gitian.zip ../../../
	popd
	./bin/gbuild --commit argentum=v${VERSION} ../argentum/contrib/gitian-descriptors/gitian-win.yml
	./bin/gsign --signer $SIGNER --release ${VERSION}-win --destination ../gitian.sigs/ ../argentum/contrib/gitian-descriptors/gitian-win.yml
	pushd build/out
	zip -r argentum-${VERSION}-win-gitian.zip *
	mv argentum-${VERSION}-win-gitian.zip ../../../
	popd
	popd

  Build output expected:

  1. linux 32-bit and 64-bit binaries + source (argentum-${VERSION}-linux-gitian.zip)
  2. windows 32-bit and 64-bit binaries + installer + source (argentum-${VERSION}-win-gitian.zip)
  3. Gitian signatures (in gitian.sigs/${VERSION}[-win]/(your gitian key)/

repackage gitian builds for release as stand-alone zip/tar/installer exe

**Linux .tar.gz:**

	unzip argentum-${VERSION}-linux-gitian.zip -d argentum-${VERSION}-linux
	tar czvf argentum-${VERSION}-linux.tar.gz argentum-${VERSION}-linux
	rm -rf argentum-${VERSION}-linux

**Windows .zip and setup.exe:**

	unzip argentum-${VERSION}-win-gitian.zip -d argentum-${VERSION}-win
	mv argentum-${VERSION}-win/argentum-*-setup.exe .
	zip -r argentum-${VERSION}-win.zip argentum-${VERSION}-win
	rm -rf argentum-${VERSION}-win

**Perform Mac build:**

  OSX binaries are created by Gavin Andresen on a 64-bit, OSX 10.6 machine.

	./autogen.sh
        SDK=$(xcode-select --print-path)/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.6.sdk
        CXXFLAGS="-mmacosx-version-min=10.6 -isysroot $SDK" ./configure --enable-upnp-default
	make
	export QTDIR=/opt/local/share/qt4  # needed to find translations/qt_*.qm files
	T=$(contrib/qt_translations.py $QTDIR/translations src/qt/locale)
        export CODESIGNARGS='--keychain ...path_to_keychain --sign "Developer ID Application: ARGENTUM FOUNDATION, INC., THE"'
	python2.7 contrib/macdeploy/macdeployqtplus Argentum-Qt.app -sign -add-qt-tr $T -dmg -fancy contrib/macdeploy/fancy.plist

 Build output expected: Argentum-Qt.dmg

###Next steps:

* Code-sign Windows -setup.exe (in a Windows virtual machine using signtool)
 Note: only Gavin has the code-signing keys currently.

* upload builds to SourceForge

* create SHA256SUMS for builds, and PGP-sign it

* update argentum.io version
  make sure all OS download links go to the right versions
  
* update forum version

* update wiki download links

* update wiki changelog: [https://en.bitcoin.it/wiki/Changelog](https://en.bitcoin.it/wiki/Changelog)

Commit your signature to gitian.sigs:

	pushd gitian.sigs
	git add ${VERSION}-linux/${SIGNER}
	git add ${VERSION}-win/${SIGNER}
	git add ${VERSION}-osx/${SIGNER}
	git commit -a
	git push  # Assuming you can push to the gitian.sigs tree
	popd

-------------------------------------------------------------------------

### After 3 or more people have gitian-built and their results match:

From a directory containing argentum source, gitian.sigs and gitian zips

	export VERSION=(new version, e.g. 0.8.0)
	mkdir argentum-${VERSION}-linux-gitian
	pushd argentum-${VERSION}-linux-gitian
	unzip ../argentum-${VERSION}-linux-gitian.zip
	mkdir gitian
	cp ../argentum/contrib/gitian-downloader/*.pgp ./gitian/
	for signer in $(ls ../gitian.sigs/${VERSION}/); do
	 cp ../gitian.sigs/${VERSION}/${signer}/argentum-build.assert ./gitian/${signer}-build.assert
	 cp ../gitian.sigs/${VERSION}/${signer}/argentum-build.assert.sig ./gitian/${signer}-build.assert.sig
	done
	zip -r argentum-${VERSION}-linux-gitian.zip *
	cp argentum-${VERSION}-linux-gitian.zip ../
	popd
	mkdir argentum-${VERSION}-win-gitian
	pushd argentum-${VERSION}-win-gitian
	unzip ../argentum-${VERSION}-win-gitian.zip
	mkdir gitian
	cp ../argentum/contrib/gitian-downloader/*.pgp ./gitian/
	for signer in $(ls ../gitian.sigs/${VERSION}-win/); do
	 cp ../gitian.sigs/${VERSION}-win/${signer}/argentum-build.assert ./gitian/${signer}-build.assert
	 cp ../gitian.sigs/${VERSION}-win/${signer}/argentum-build.assert.sig ./gitian/${signer}-build.assert.sig
	done
	zip -r argentum-${VERSION}-win-gitian.zip *
	cp argentum-${VERSION}-win-gitian.zip ../
	popd

    - Code-sign MacOSX .dmg

  Note: only Gavin has the code-signing keys currently.

- Create `SHA256SUMS.asc` for builds, and PGP-sign it. This is done manually.
  Include all the files to be uploaded. The file has `sha256sum` format with a
  simple header at the top:

```
Hash: SHA256

0060f7d38b98113ab912d4c184000291d7f026eaf77ca5830deec15059678f54  bitcoin-x.y.z-linux.tar.gz
...
```

- Upload gitian zips to SourceForge

- Announce the release:

  - Add the release to argentum.io

  - Announce on reddit /r/argentum, /r/argentumdev

  - Release sticky on discuss argentum: https://www.github.com/argentumproject/

- Celebrate 
