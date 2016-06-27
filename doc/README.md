Fedoracoin 0.9.0 BETA
=====================

Copyright (c) 2009-2014 Fedoracoin Developers


Setup
---------------------
[Fedoracoin Core] is the original Fedoracoin client and it builds the backbone of the network. However, it downloads and stores the entire history of Fedoracoin transactions (which is currently several GBs); depending on the speed of your computer and network connection, the synchronization process can take anywhere from a few hours to a day or more. Thankfully you only have to do this once. 

Running
---------------------
The following are some helpful notes on how to run Fedoracoin on your native platform. 

### Unix

You need the Qt5 run-time libraries to run Fedoracoin-Qt.
You also need the zxing (Zebra Crossing) libraries for qrcode support.
The libraries should be installed by your package manager,if you use an installation package for you distribution.

If you need to compile or install the zxing libraries yourself, download the sources from github.com.
Search for zxing-cpp to get the c++ version of the code.





### Windows

Unpack the files into a directory, and then run fedoracoin-qt.exe.

### OSX

Drag Fedoracoin-Qt to your applications folder, and then run Fedoracoin-Qt.

### Need Help?

* Ask for support in the support section of https://forum.fedoracoin.com
* If you find a bug, you also may raise an issue at: http://www.github.com/Fedoracoin/Fedoracoin/issues

Building
---------------------
The developer documentation can be found on http://www.github.com/Fedoracoin/Fedoracoin/tree/0.9.3/doc 

check for 

- [OSX Build Notes](build-osx.md)
- [Unix Build Notes](build-unix.md)
- [Windows Build Notes](build-msw.md)

Development
---------------------
The Fedoracoin repo's [root README](https://github.com/Fedoracoin/Fedoracoin/tree/0.9.3/README.md) contains relevant information on the development process and automated testing.

- [Coding Guidelines](coding.md)
- [Multiwallet Qt Development](multiwallet-qt.md)
- [Release Notes](release-notes.md)
- [Release Process](release-process.md)
- [Source Code Documentation (External Link)](https://dev.visucore.com/fedoracoin/doxygen/)
- [Translation Process](translation_process.md)
- [Unit Tests](unit-tests.md)

### Resources
- forum.fedoracoin.comp
- chat: forum.fedoracoin.com/shoutbox


License
---------------------
Distributed under the [MIT/X11 software license](http://www.opensource.org/licenses/mit-license.php).
This product includes software developed by the OpenSSL Project for use in the [OpenSSL Toolkit](http://www.openssl.org/). This product includes
cryptographic software written by Eric Young ([eay@cryptsoft.com](mailto:eay@cryptsoft.com)), and UPnP software written by Thomas Bernard.
