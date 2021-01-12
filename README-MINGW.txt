
# Bullseye/testing amd64 qemu VM 2021-01-05

apt update
apt dist-upgrade

apt install systemd-coredump mc git pkgconf cmake mingw-w64 python3-distutils







$ dpkg -l | grep mingw
ii  binutils-mingw-w64-i686            2.35-2+8.11                    amd64        Cross-binutils for Win32 (x86) using MinGW-w64
ii  binutils-mingw-w64-x86-64          2.35-2+8.11                    amd64        Cross-binutils for Win64 (x64) using MinGW-w64
ii  g++-mingw-w64                      10.2.0-18+24                   all          GNU C++ compiler for MinGW-w64
ii  g++-mingw-w64-i686                 10.2.0-18+24                   all          GNU C++ compiler for MinGW-w64 targeting Win32
ii  g++-mingw-w64-i686-posix           10.2.0-19+24+b1                amd64        GNU C++ compiler for MinGW-w64, Win32/POSIX
ii  g++-mingw-w64-i686-win32           10.2.0-19+24+b1                amd64        GNU C++ compiler for MinGW-w64, Win32/Win32
ii  g++-mingw-w64-x86-64               10.2.0-18+24                   all          GNU C++ compiler for MinGW-w64 targeting Win64
ii  g++-mingw-w64-x86-64-posix         10.2.0-19+24+b1                amd64        GNU C++ compiler for MinGW-w64, Win64/POSIX
ii  g++-mingw-w64-x86-64-win32         10.2.0-19+24+b1                amd64        GNU C++ compiler for MinGW-w64, Win64/Win32
ii  gcc-mingw-w64                      10.2.0-18+24                   all          GNU C compiler for MinGW-w64
ii  gcc-mingw-w64-base                 10.2.0-19+24+b1                amd64        GNU Compiler Collection for MinGW-w64 (base package)
ii  gcc-mingw-w64-i686                 10.2.0-18+24                   all          GNU C compiler for MinGW-w64 targeting Win32
ii  gcc-mingw-w64-i686-posix           10.2.0-19+24+b1                amd64        GNU C compiler for MinGW-w64, Win32/POSIX
ii  gcc-mingw-w64-i686-posix-runtime   10.2.0-19+24+b1                amd64        GNU Compiler Collection for MinGW-w64, i686/posix
ii  gcc-mingw-w64-i686-win32           10.2.0-19+24+b1                amd64        GNU C compiler for MinGW-w64, Win32/Win32
ii  gcc-mingw-w64-i686-win32-runtime   10.2.0-19+24+b1                amd64        GNU Compiler Collection for MinGW-w64, i686/win32
ii  gcc-mingw-w64-x86-64               10.2.0-18+24                   all          GNU C compiler for MinGW-w64 targeting Win64
ii  gcc-mingw-w64-x86-64-posix         10.2.0-19+24+b1                amd64        GNU C compiler for MinGW-w64, Win64/POSIX
ii  gcc-mingw-w64-x86-64-posix-runtime 10.2.0-19+24+b1                amd64        GNU Compiler Collection for MinGW-w64, x86-64/posix
ii  gcc-mingw-w64-x86-64-win32         10.2.0-19+24+b1                amd64        GNU C compiler for MinGW-w64, Win64/Win32
ii  gcc-mingw-w64-x86-64-win32-runtime 10.2.0-19+24+b1                amd64        GNU Compiler Collection for MinGW-w64, x86-64/win32
ii  mingw-w64                          8.0.0-1                        all          Development environment targeting 32- and 64-bit Windows
ii  mingw-w64-common                   8.0.0-1                        all          Common files for Mingw-w64
ii  mingw-w64-i686-dev                 8.0.0-1                        all          Development files for MinGW-w64 targeting Win32
ii  mingw-w64-x86-64-dev               8.0.0-1                        all          Development files for MinGW-w64 targeting Win64












# -posix for libstdc++'s C++11 <thread>, <mutex>, and <future>

mkdir /home/benutzer/source
cat <<EOF > /home/benutzer/source/Toolchain-Debian-mingw64-posix.cmake
# the name of the target operating system
SET(CMAKE_SYSTEM_NAME Windows)

# which compilers to use for C and C++
SET(CMAKE_C_COMPILER x86_64-w64-mingw32-gcc-posix)
SET(CMAKE_CXX_COMPILER x86_64-w64-mingw32-g++-posix)
SET(CMAKE_RC_COMPILER x86_64-w64-mingw32-windres)

# here is the target environment located
#SET(CMAKE_FIND_ROOT_PATH  /usr/x86_64-w64-mingw32 /home/alex/mingw-install )

# adjust the default behaviour of the FIND_XXX() commands:
# search headers and libraries in the target environment, search
# programs in the host environment
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
EOF









export MYPREFIX=/home/benutzer/source/mingw-prefix
export PKG_CONFIG_PATH=$MYPREFIX/lib/pkgconfig







mkdir /home/benutzer/source/openssl -p
cd    /home/benutzer/source/openssl
wget https://www.openssl.org/source/openssl-1.1.1i.tar.gz
tar -zxf openssl-1.1.1i.tar.gz
cd openssl-1.1.1i
./Configure --prefix=$MYPREFIX --cross-compile-prefix=x86_64-w64-mingw32- mingw64
make -j16
make install









mkdir /home/benutzer/source/libusb -p
cd    /home/benutzer/source/libusb
wget https://github.com/libusb/libusb/releases/download/v1.0.24/libusb-1.0.24.tar.bz2
tar -jxf libusb-1.0.24.tar.bz2
cd libusb-1.0.24
./configure --prefix=$MYPREFIX --build=x86_64-linux-gnu --host=x86_64-w64-mingw32
make -j16
make install














mkdir /home/benutzer/source/zlib -p
cd    /home/benutzer/source/zlib
wget http://zlib.net/zlib-1.2.11.tar.xz
tar -axf zlib-1.2.11.tar.xz
mkdir obj
cd    obj
cmake \
    -DCMAKE_TOOLCHAIN_FILE=/home/benutzer/source/Toolchain-Debian-mingw64-posix.cmake \
    -DCMAKE_INSTALL_PREFIX=$MYPREFIX \
    ../zlib-1.2.11
make -j16
make install
ln -s /home/benutzer/source/mingw-prefix/share/pkgconfig/zlib.pc /home/benutzer/source/mingw-prefix/lib/pkgconfig/zlib.pc
sed -i 's/-lz/-lzlib/g' /home/benutzer/source/mingw-prefix/lib/pkgconfig/zlib.pc








mkdir /home/benutzer/source/libpng -p
cd    /home/benutzer/source/libpng
wget "https://downloads.sourceforge.net/project/libpng/libpng16/1.6.37/libpng-1.6.37.tar.xz?r=https%3A%2F%2Fsourceforge.net%2Fprojects%2Flibpng%2Ffiles%2Flibpng16%2F1.6.37%2Flibpng-1.6.37.tar.xz%2Fdownload%3Fuse_mirror%3Dkumisystems&ts=1610297671" -O libpng-1.6.37.tar.xz
tar axf libpng-1.6.37.tar.xz
mkdir obj
cd    obj
cmake \
    -DCMAKE_TOOLCHAIN_FILE=/home/benutzer/source/Toolchain-Debian-mingw64-posix.cmake \
    -DCMAKE_INSTALL_PREFIX=$MYPREFIX \
    ../libpng-1.6.37
make -j16
make install
sed -i 's/-lz/-lzlib/g' /home/benutzer/source/mingw-prefix/lib/pkgconfig/libpng.pc







mkdir /home/benutzer/source/pixman -p
cd    /home/benutzer/source/pixman
wget https://cairographics.org/releases/pixman-0.40.0.tar.gz
tar -zxf pixman-0.40.0.tar.gz
cd pixman-0.40.0
./configure --prefix=$MYPREFIX --build=x86_64-linux-gnu --host=x86_64-w64-mingw32
make -j16
make install








mkdir /home/benutzer/source/cairo -p
cd    /home/benutzer/source/cairo
wget https://cairographics.org/releases/cairo-1.16.0.tar.xz
tar -axf cairo-1.16.0.tar.xz
cd cairo-1.16.0
# pkg-config --cflags --libs zlib
export CFLAGS=-I/home/benutzer/source/mingw-prefix/include
export LDFLAGS=-L/home/benutzer/source/mingw-prefix/lib
sed -i 's/LIBS="-lz  $LIBS"/LIBS="-lzlib  $LIBS"/g' configure
sed -i 's/-lz$/-lzlib/g' configure
sed -i 's/-lz$/-lzlib/g' ./util/cairo-trace/Makefile.am
sed -i 's/-lz$/-lzlib/g' ./util/cairo-script/Makefile.am
./configure --prefix=$MYPREFIX --build=x86_64-linux-gnu --host=x86_64-w64-mingw32 --enable-test-surfaces=no --enable-full-testing=no
touch test/cairo_test_suite-font-variations.o
make -j16
make install
unset CFLAGS
unset LDFLAGS









mkdir /home/benutzer/source/mbedtls -p
cd    /home/benutzer/source/mbedtls
wget https://github.com/ARMmbed/mbedtls/archive/v2.25.0.tar.gz
tar -zxf v2.25.0.tar.gz
mkdir obj
cd    obj
cmake \
    -DCMAKE_TOOLCHAIN_FILE=/home/benutzer/source/Toolchain-Debian-mingw64-posix.cmake \
    -DCMAKE_INSTALL_PREFIX=$MYPREFIX \
    ../mbedtls-2.25.0
make -j16
make install











mkdir /home/benutzer/source/libjpeg -p
cd    /home/benutzer/source/libjpeg
wget "https://downloads.sourceforge.net/project/libjpeg-turbo/2.0.6/libjpeg-turbo-2.0.6.tar.gz?r=https%3A%2F%2Fsourceforge.net%2Fprojects%2Flibjpeg-turbo%2Ffiles%2F2.0.6%2Flibjpeg-turbo-2.0.6.tar.gz%2Fdownload&ts=1610325477" -O libjpeg-turbo-2.0.6.tar.gz
tar -zxf libjpeg-turbo-2.0.6.tar.gz
mkdir obj
cd    obj
# Change this line in CMakeLists.txt:
# string(TOLOWER "${CMAKE_SYSTEM_PROCESSOR}" CMAKE_SYSTEM_PROCESSOR_LC)
cmake \
    -DCMAKE_TOOLCHAIN_FILE=/home/benutzer/source/Toolchain-Debian-mingw64-posix.cmake \
    -DCMAKE_INSTALL_PREFIX=$MYPREFIX \
    -DREQUIRE_SIMD=FALSE \
    -DWITH_SIMD=FALSE \
    -DCPU_TYPE=x86_64 \
    ../libjpeg-turbo-2.0.6
make -j16
make install











mkdir /home/benutzer/source/freerdp/git -p
cd    /home/benutzer/source/freerdp/git
git clone https://github.com/FreeRDP/FreeRDP.git

mv FreeRDP/client/Windows/wfreerdp.rc /tmp/wfreerdp.rc
# Convert wfreerdp.rc from UTF-16 to UTF-8
iconv -f UTF-16LE -t UTF-8 /tmp/wfreerdp.rc -o FreeRDP/client/Windows/wfreerdp.rc
# Remove BOM
sed -i 's/\xef\xbb\xbf//' FreeRDP/client/Windows/wfreerdp.rc
# Convert double backslash to slash.
sed -i 's@\\\\@/@g' FreeRDP/client/Windows/wfreerdp.rc

mkdir obj
cd    obj



# https://gitlab.kitware.com/cmake/community/-/wikis/doc/cmake/cross_compiling/Mingw




cmake \
    -DCMAKE_TOOLCHAIN_FILE=/home/benutzer/source/Toolchain-Debian-mingw64-posix.cmake \
    -DOPENSSL_ROOT_DIR=$MYPREFIX \
    -DLIBUSB_1_INCLUDE_DIR=$MYPREFIX/include/libusb-1.0 \
    -DLIBUSB_1_LIBRARY=$MYPREFIX/lib/libusb-1.0.dll.a \
    -DMBEDTLS_INCLUDE_DIR=$MYPREFIX/include \
    -DMBEDCRYPTO_LIBRARY=$MYPREFIX/lib/libmbedcrypto.a \
    -DMBEDTLS_LIBRARY=$MYPREFIX/lib/libmbedtls.a \
    -DMBEDX509_LIBRARY=$MYPREFIX/lib/libmbedx509.a \
    -DWITH_MBEDTLS=ON \
    -DCAIRO_INCLUDE_DIR=/home/benutzer/source/mingw-prefix/include/cairo \
    -DWITH_CAIRO=ON \
    -DWITH_JPEG=ON \
    -DCMAKE_INSTALL_PREFIX=$MYPREFIX \
    ../FreeRDP

make -j16
