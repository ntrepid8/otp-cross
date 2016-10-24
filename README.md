# How to cross compile Erlang OTP including crypto and ncurses

Forked from [tonyrog/otp-cross](https://github.com/tonyrog/otp-cross) with updates for
building for a `armv7-unknown-linux-musleabihf` target.

See also:
- <http://erlang.org/documentation/doc-5.7.5/doc/installation_guide/source/INSTALL-CROSS.html>

First get install a cross compiler. Here I will use armv7-unknown-linux-musleabihf.

Then follow the instructions below

## zlib
    > wget http://zlib.net/zlib-1.2.8.tar.gz
    > tar xf zlib-1.2.8.tar.gz
    > cd zlib-1.2.8
    > export CC=arm-linux-gnueabi-gcc
    > ./configure --prefix=$HOME/arm
    > make
    > make install
    > unset CC  # thanks benoitc

## openssl
    > wget http://openssl.org/source/openssl-1.0.2j.tar.gz
    > tar xf openssl-1.0.2f.tar.gz
    > export cross=arm-linux-gnueabi-
    > cd openssl-1.0.2f
    > ./Configure dist --prefix=$HOME/arm
    > make CC="${cross}gcc" AR="${cross}ar r" RANLIB="${cross}ranlib"
    > make install

## ncurses
    > wget http://ftp.gnu.org/pub/gnu/ncurses/ncurses-6.0.tar.gz
    > tar xf ncurses-6.0.tar.gz
    > cd ncurses-6.0
    > ./configure --host=arm-linux-gnueabi --prefix=$HOME/arm --without-ada --without-cxx --without-cxx-binding --without-manpages --without-progs --without-tests CPPFLAGS="-P"
    > make
    > make install

## otp
    > wget http://www.erlang.org/download/otp_src_18.2.1.tar.gz
    > tar xf otp_src_18.2.1.tar.gz
    > cd otp_src_18.2.1

Now modify xcomp/erl-xcomp-arm-linux.conf, replace arm-wrs-linux-gnueabi with arm-linux-gnueabi

    > ./otp_build configure 
    > export ARM_SYSROOT=$HOME/arm
    > ./otp_build configure --with-ssl=$HOME/arm --disable-dynamic-ssl-lib --xcomp-conf=./xcomp/erl-xcomp-arm-linux.conf
    > ./otp_build boot -a
    > ./otp_build release -a $HOME/arm/erlang

When building the tar file we remove some apps in order to reduce
the size of the image. We could probably add --exclude=erlang/lib/*/src*
as well.

    tar czf erlang.tgz --exclude=erlang/lib/cos* --exclude=erlang/lib/common_test* --exclude=erlang/lib/percept* --exclude=erlang/lib/eunit* --exclude=erlang/lib/gs* --exclude=erlang/lib/ic* --exclude=erlang/lib/jinterface* --exclude=erlang/lib/megaco* --exclude=erlang/lib/test_server* --exclude=erlang/lib/dialyzer* --exclude=erlang/lib/observer* --exclude=erlang/lib/diameter* --exclude=erlang/lib/orber* --exclude=erlang/lib/typer* --exclude=erlang/lib/edoc* --exclude=erlang/lib/ose* --exclude=erlang/lib/webtool* --exclude=erlang/lib/eldap* --exclude=erlang/lib/wx* --exclude=erlang/lib/erl_docgen* --exclude=erlang/lib/otp_mibs* ./erlang

Here is how to install the erlang.tgz on the target system

    scp erlang.tgz user@ip:

then ssh/minicom to the target and

    cd /usr/lib
    tar xf ~user/erlang.tgz
    cd erlang
    ./Install -minimal /usr/lib/erlang
    ln -s /usr/lib/erlang/bin/erl /usr/bin/erl

To get termcap working ( the ncurses step ) some data located under
$HOME/arm/share/terminfo needs to be (re)located on the target under
for example $HOME/.terminfo. Or you may set a symbolic link from
$HOME/.terminfo  -> /usr/share/terminfo 
Ncurses may have options to fix this at build time ?

# How to build applications

Use rebar and set the REBAR_TARGET_ARCH. This will cross compile
most rebar applications that contain driver/nif source code.

    cd my_app
    REBAR_TARGET_ARCH=arm-linux-gnueabi rebar compile
