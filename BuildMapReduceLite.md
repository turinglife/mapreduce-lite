

# Introduction #

MapReduce Lite can be built on Linux, Mac OS X and FreeBSD, with GCC (>=4.5.1) or Clang (>=3.0).

MapReduce Lite depends on the following tools or libraries:

  * [CMake](http://www.cmake.org/) is used to manage and build the project.
  * [Google Protocol Buffers](http://code.google.com/p/protobuf/) is used for inter-worker communication and disk storage.
  * [GFlags](http://code.google.com/p/google-gflags/) is used to parse command line options.
  * [Google Test](http://code.google.com/p/googletest/) is used to write and run unit tests.
  * [Boost](http://www.boost.org/) helps developing multi-threading in asynchronous inter-worker communication and provides cross-platform filesystem support.
  * [libevent](http://libevent.org) is used to support efficient asynchronous network communication.

Historically, MapReduce Lite relies on Linux-specific system call, [epoll](http://linux.die.net/man/4/epoll), which prevented MapReduce Lite from being built and run on MacOS X and Cygwin.  Hao Yan has changed the code recently (2013-10-03) to replace the use of epoll by `libevent` (see [this issue](http://code.google.com/p/mapreduce-lite/issues/detail?id=2)).

# Install Building Tools #

## Install GCC or Clang ##

MapReduce Lite can be built using GCC or Clang.

  * On Cygwin, run `setup.exe` and install `gcc` and `binutils`.
  * On Debian/Ubuntu Linux, type the command `sudo apt-get install gcc binutils` to install GCC, or `sudo apt-get install clang` to install Clang.
  * On FreeBSD, type the command `sudo pkg_add -r clang` to install Clang.  Note that since version 9.0, FreeBSD does not update GCC but relies completely on Clang.
  * On Mac OS X, install XCode gets you Clang.

## Install CMake ##

MapReduce Lite need CMake with version >= 2.8.0 to compile Google Protocol Buffer definitions.

To install CMake from binary packages:
  * On Cygwin, run `setup.exe` and install `cmake`.
  * On Debian/Ubuntu Linux, type the command `sudo apt-get install cmake`.
  * On FreeBSD, type the command `sudo pkg_add -r cmake`.
  * On Mac OS X, if you have [Howebew](http://mxcl.github.com/homebrew/), you can use the command `brew install cmake`, or if you have [MacPorts](http://www.macports.org/), run `sudo port install cmake`.  You won't want to have both Homebrew and MacPorts installed.

You can also download binary or source package of CMake from: http://www.cmake.org/cmake/resources/software.html.

## Install Protobuf ##

MapReduce Lite requires protobuf with version >= 2.3.0.

To install protobuf from binary packages:
  * On Debian/Ubuntu Linux, you can run `sudo apt-get install libprotobuf-dev libprotoc-dev`.
  * On FreeBSD, you can run `sudo pkg_add -r protobuf`.
  * On Mac OS X, you can run `brew install protobuf protobuf-c`.

Or, you can install protobuf from source code:
  1. Download source code package, say `protobuf-2.5.0.tar.bz2`, from http://code.google.com/p/protobuf
  1. You need to install protobuf into a standard place, e.g., `/usr/local/`, so that CMake can find the protoc compiler and the library:
```
tar xjf protobuf-2.5.0.tar.bz2
cd protobuf-2.5.0
./configure --disable-shared --enable-static
make
sudo make install
```

# Install Dependencies #

It is recommended to install dependent libraries from source code, so you have more control on how to build them.  For example, if you want to build MapReduce Lite using Clang, you need to build these dependencies using Clang too.

## Install GFlags ##

  1. Download the source code package (e.g., `gflags-2.0.tar.gz`) from http://code.google.com/p/google-gflags
  1. Unpack the source code anywhere (e.g., `./gflags-2.0`)
  1. You are free to install gflags anywhere (e.g., `/home/you/3rd-party/gflags`):
```
cd gflags-2.0
./configure --prefix=/home/you/3rd-party/gflags-2.0 # if you want to use default build tool, or
# CC=clang CXX=clang++ ./configure --prefix=/home/you/3rd-party/gflags-2.0
make && make install
ln -s /home/you/3rd-party/gflags-2.0 /home/you/3rd-party/gflags
```

## Install GoogleTest ##

  1. Download the source code package (e.g., `gtest-1.7.0.tar.bz2`) from http://code.google.com/p/googletest/.
  1. Unpack the source code, say, to `/home/you/3rd-party/gtest-1.7.0`
  1. Make a symbolic link
```
ln -s /home/you/3rd-party/gtest-1.7.0 /home/you/3rd-party/gtest
```

Please note that since version 1.6.0, there is no longer `configure; make; make install` in google-test; instead, you have to incorporate the source code into your projects.  As google-test provides a CMakeLists.txt file, the CMakeLists.txt of MapReduce Lite simply includes that file.

## Install Boost ##

We need a recent version of Boost.  Download the source package from http://www.boost.org/users/download/, and

```
cd /home/you/3rd-party/
tar xjf boost_1_54_0.tar.bz2
cd boost_1_54_0
./bootstrap --prefix=/home/you/3rd-party/boost-1_54_0
./b2 -j8 # if you want to use default build tool, or
# ./b2 -j8 toolset=clang
ln -s /home/you/3rd-party/boost-1_54_0 /home/you/3rd-party/boost
```

## Install libevent ##

  1. Download libevent from http://libevent.org
  1. Unpack the source code, build and install:
```
cd /home/you/3rd-party
tar xjf libevent-2.0.21-stable.tar.bz2
cd libevent-2.0.21-stable
./configure --prefix=/home/you/3rd-party/libevent-2.0.21-stable # or
# CC=clang CXX=clang++ ./configure --prefix=/home/you/3rd-party/libevent-2.0.21-stable
make && make install
ln -s /home/you/3rd-party/libevent-2.0.21-stable /home/wyi/3rd-party/libevent
```

# Build MapReduce Lite #

With above dependencies installed, building MapReduce Lite is easy:

  1. Checkout the code:
```
svn checkout http://mapreduce-lite.googlecode.com/svn/trunk/ mapreduce-lite-read-only
```
  1. Open `mapreduce-lite-read-only/CMakeLists.txt` and replace the value in the following line by the directory where you put third-party libraries.
```
set(THIRD_PARTY_DIR "/home/you/3rd-party")
```
  1. Replace the value in the following line by the directory where you want to install MapReduce Lite and demos.
```
set(CMAKE_INSTALL_PREFIX "/home/you/mapreduce-lite")
```
  1. Build MapReduce Lite
```
mkdir build
cd build
cmake ../mapreduce-lite-read-only # if you want to use default build tools, or
# CXX=clang++ CC=clang cmake ../mapreduce-lite-read-only
# CXX=g++     CC=gcc   cmake ../mapreduce-lite-read-only
make
make install
```