# Qt6 Cross-Compilation Guide for Raspberry Pi

This guide provides step-by-step instructions for cross-compiling Qt6 for Raspberry Pi 4 ModelB 8 GB using Ubuntu 22.04. It covers the entire process from preparing the Raspberry Pi and the host system to building Qt and testing a sample application. It's derived from MuyePan's [Cross compilation of Qt6.5.1 for RPI](https://github.com/MuyePan/CrossCompileQtForRpi)

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Prepare Raspberry Pi](#prepare-raspberry-pi)
3. [Prepare Host System](#prepare-host-system)
4. [Build GCC as a Cross Compiler](#build-gcc-as-a-cross-compiler)
5. [Building Qt6](#building-qt6)
6. [Setting Up Qt Creator](#setting-up-qt-creator)
7. [Testing HelloWorld](#testing-helloworld)
8. [Adding QML Module](#adding-qml-module)
9. [Testing HelloWorldQml](#testing-helloworldqml)
10. [Adding Serial Bus Module](#adding-serial-bus-module)
11. [Troubleshooting](#troubleshooting)
12. [References](#references)

## Prerequisites

- A Linux-based host machine. (Ubuntu 22.04)
- Raspberry Pi running Raspbian or a compatible OS. (Raspian OS recommended)
- Sufficient storage and memory on both systems.
- Basic knowledge of Linux command line
- CMake and build system basic knowledge

## Prepare Raspberry Pi

Install the 64-bit Raspberry Pi OS with desktop.

In this tutorial used version: Raspberry Pi OS (Legacy) with desktop and recommended softwares image: [Raspberry Pi OS (Legacy, 64-bit) Debian version: 11 (bullseye)](https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-legacy-64-bit)

You can write to your SD Disk with [Raspberry PI Imager](https://www.raspberrypi.com/software/).


1. Install the latest 64-bit Raspberry Pi OS with desktop and update the system:

    ```bash
    sudo apt update
    sudo apt upgrade
    sudo reboot
    ```

2. Ensure SSH is Installed and Enabled on the RPi and take note RPi's IP Address.

    ```bash
    sudo raspi-config
    ```
- Navigate to Interfacing Options > SSH and enable it.
- Alternatively, if you can’t access the Raspberry Pi's terminal directly, place an empty file named ssh (no extension) in the /boot/ directory of the SD card. This will enable SSH on boot.
- To get RPi's IP Address 

    ```bash
    hostname -I
    ```

3. Install necessary packages:

    ```bash
    sudo apt-get install libboost-all-dev libudev-dev libinput-dev libts-dev libmtdev-dev libjpeg-dev libfontconfig1-dev libssl-dev libdbus-1-dev libglib2.0-dev libxkbcommon-dev libegl1-mesa-dev libgbm-dev libgles2-mesa-dev mesa-common-dev libasound2-dev libpulse-dev gstreamer1.0-omx libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev  gstreamer1.0-alsa libvpx-dev libsrtp2-dev libsnappy-dev libnss3-dev "^libxcb.*" flex bison libxslt-dev ruby gperf libbz2-dev libcups2-dev libatkmm-1.6-dev libxi6 libxcomposite1 libfreetype6-dev libicu-dev libsqlite3-dev libxslt1-dev 
    ```

    If you are using **bullseye** keep **libgst-dev** package, if you are using **bookworm** please remove **libgst-dev** package. Otherwise you'll get an error *Qt6: Unable to locate package libgst-dev*

    ```bash
    sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libx11-dev freetds-dev libsqlite3-dev libpq-dev libiodbc2-dev firebird-dev libgst-dev libxext-dev libxcb1 libxcb1-dev libx11-xcb1 libx11-xcb-dev libxcb-keysyms1 libxcb-keysyms1-dev libxcb-image0 libxcb-image0-dev libxcb-shm0 libxcb-shm0-dev libxcb-icccm4 libxcb-icccm4-dev libxcb-sync1 libxcb-sync-dev libxcb-render-util0 libxcb-render-util0-dev libxcb-xfixes0-dev libxrender-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-glx0-dev libxi-dev libdrm-dev libxcb-xinerama0 libxcb-xinerama0-dev libatspi2.0-dev libxcursor-dev libxcomposite-dev libxdamage-dev libxss-dev libxtst-dev libpci-dev libcap-dev libxrandr-dev libdirectfb-dev libaudio-dev libxkbcommon-x11-dev gdbserver
    ```
4. Take note the versions of GCC, LD and LDD. Source code of the same version should be downloaded to build cross compiler. We need this versions in [Build GCC as a Cross Compiler](#build-gcc-as-a-cross-compiler) section

    ```bash
    gcc --version
    ld --version
    ldd --version
    ```
5. Create a folder for Qt6 installation:

    ```bash
    sudo mkdir /usr/local/qt6
    sudo chmod 777 /usr/local/bin
    ```

6. Append the following to `~/.bashrc`:

    ```bash
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/qt6/lib/
    ```

7. Update the changes:

    ```bash
    source ~/.bashrc
    ```

## Prepare Host System

1. Create a virtual machine for Ubuntu 22.04 and update the system: You can download image: [Ubuntu 22.04.4 LTS (Jammy Jellyfish) 64-bit PC (AMD64) desktop image](https://releases.ubuntu.com/jammy/)
For Virtual Machine creation you can use [Virtual Box](https://www.virtualbox.org/wiki/Downloads) or [VMware Workstation Pro for Window and Linux or VMWare Fusion for Mac](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion).

    ```bash
    sudo apt update
    sudo apt upgrade
    ```
2. Install necessary packages:

    ```bash
    sudo apt-get install make build-essential libclang-dev ninja-build gcc git bison python3 gperf pkg-config libfontconfig1-dev libfreetype6-dev libx11-dev libx11-xcb-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libxcb-glx0-dev libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-util-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev libatspi2.0-dev libgl1-mesa-dev libglu1-mesa-dev freeglut3-dev build-essential gawk git texinfo bison file wget libssl-dev gdbserver gdb-multiarch libxcb-cursor-dev
    ```

3. Build the latest CMake from source:

    ```bash
    cd ~
    git clone https://github.com/Kitware/CMake.git
    cd CMake
    ./bootstrap && make -j8 && sudo make install
    ```

4. Check the CMake version to verify installed properly.

    ```bash
    cmake --version
    ```

5. CMake folder is not need any more. We can delete it.
    ```bash
    sudo rm -rf ~/CMake
    ```

6. Connect to RPi with SSH Key Pair.

- Use the ssh command from your Ubuntu terminal: Replace `<RPI_IP>` with your Raspberry Pi's IP address

    ```bash
    ssh pi@<RPI_IP>
    ```
- The default username is **pi**, and the default password is **raspberry**. Change the password after logging in for security (optional):

    ```bash
    passwd
    ```
- Generate an SSH Key Pair on Ubuntu (Host Machine)

    ```bash
    ssh-keygen -t rsa -b 4096
    ```
- Press Enter to accept the default file location (/home/your_username/.ssh/id_rsa).

- Optionally, set a passphrase for additional security.

- Copy Your SSH Public Key to the Raspberry Pi: To enable password-less login, copy your SSH public key to the Raspberry Pi. Replace `<RPI_IP>` with your Raspberry Pi's IP address

    ```bash
    ssh-copy-id pi@<RPI_IP>
    ```
    You’ll need to enter the password for the pi user during this process.

- Connect Using SSH Key Authentication. Now, you should be able to connect to the Raspberry Pi without entering a password. Replace `<RPI_IP>` with your Raspberry Pi's IP address

    ```bash
    ssh pi@<RPI_IP>
    ```

## Build GCC as a Cross Compiler

**You should modify the following commands to your needs.** In previous steps we took as note of GCC, LD and LDD versions. Now we should download source code according to this versions. If your version does not exist use the closest one.

1. Download necessary source code:

    ```bash
    cd ~
    mkdir gcc_all && cd gcc_all
    wget https://ftpmirror.gnu.org/binutils/binutils-2.35.2.tar.bz2
    wget https://ftpmirror.gnu.org/glibc/glibc-2.31.tar.bz2
    wget https://ftpmirror.gnu.org/gcc/gcc-10.3.0/gcc-10.3.0.tar.gz
    git clone --depth=1 https://github.com/raspberrypi/linux
    tar xf binutils-2.35.2.tar.bz2
    tar xf glibc-2.31.tar.bz2
    tar xf gcc-10.3.0.tar.gz
    rm *.tar.*
    cd gcc-10.3.0
    contrib/download_prerequisites
    ```

2. Make a folder for the compiler installation:

    ```bash
    sudo mkdir -p /opt/cross-pi-gcc
    sudo chown $USER /opt/cross-pi-gcc
    export PATH=/opt/cross-pi-gcc/bin:$PATH
    ```

3. Copy the kernel headers: These are the Linux kernel header files that define the interface between the kernel and user space. They contain important definitions and structures that are necessary for compiling programs that interact with the kernel. hen cross-compiling, we're building software on one system (the host) that will run on a different system (the target, in this case, the Raspberry Pi). The compiler needs to know about the target system's kernel interface. This step is part of creating a complete cross-compilation toolchain. The headers complement the cross-compiler (GCC) and cross-library (Glibc) to form a full development environment for the target system. 32-bit users can use **KERNEL=kernel7**

    ```bash
    cd ~/gcc_all
    cd linux
    KERNEL=kernel8
    make ARCH=arm64 INSTALL_HDR_PATH=/opt/cross-pi-gcc/aarch64-linux-gnu headers_install
    ```

4. Build Binutils:  **You should modify the following commands to your needs.**

    ```bash
    cd ~/gcc_all
    mkdir build-binutils && cd build-binutils
    ../binutils-2.35.2/configure --prefix=/opt/cross-pi-gcc --target=aarch64-linux-gnu --with-arch=armv8 --disable-multilib
    make -j 8
    make install
    ```

5. Edit `gcc-10.3.0/libsanitizer/asan/asan_linux.cpp` (**You should modify the path according to your version**). Add the following:

    ```cpp
    #ifndef PATH_MAX
    #define PATH_MAX 4096
    #endif
    ```

6. Do a partial build of GCC: **You should modify the following commands to your needs.**

    ```bash
    cd ~/gcc_all
    mkdir build-gcc && cd build-gcc
    ../gcc-10.3.0/configure --prefix=/opt/cross-pi-gcc --target=aarch64-linux-gnu --enable-languages=c,c++ --disable-multilib
    make -j8 all-gcc
    make install-gcc
    ```
 
7. Partially build Glibc: **You should modify the following commands to your needs.**

    ```bash
    cd ~/gcc_all
    mkdir build-glibc && cd build-glibc
    ../glibc-2.31/configure --prefix=/opt/cross-pi-gcc/aarch64-linux-gnu --build=$MACHTYPE --host=aarch64-linux-gnu --target=aarch64-linux-gnu --with-headers=/opt/cross-pi-gcc/aarch64-linux-gnu/include --disable-multilib libc_cv_forced_unwind=yes
    make install-bootstrap-headers=yes install-headers
    make -j8 csu/subdir_lib
    install csu/crt1.o csu/crti.o csu/crtn.o /opt/cross-pi-gcc/aarch64-linux-gnu/lib
    aarch64-linux-gnu-gcc -nostdlib -nostartfiles -shared -x c /dev/null -o /opt/cross-pi-gcc/aarch64-linux-gnu/lib/libc.so
    touch /opt/cross-pi-gcc/aarch64-linux-gnu/include/gnu/stubs.h
    ```

8. Finish building GCC:

    ```bash
    cd ~/gcc_all/build-gcc
    make -j8 all-target-libgcc
    make install-target-libgcc
    ```

9. Finish building Glibc:

    ```bash
    cd ~/gcc_all/build-glibc
    make -j8
    make install
    ```

10. Complete GCC build:

    ```bash
    cd ~/gcc_all/build-gcc
    make -j8
    make install
    ```

    At this point, we have a full cross compiler toolchain with GCC. Folder gcc_all is not need any more. You can delete it.

    ```bash
    sudo rm -rf ~/gcc_all
    ```

## Building Qt6

1. Make folders for sysroot and Qt6:

    ```bash
    cd ~
    mkdir rpi-sysroot rpi-sysroot/usr rpi-sysroot/opt
    mkdir qt6 qt6/host qt6/pi qt6/host-build qt6/pi-build qt6/src
    ```

2. Download QtBase source code:

    ```bash
    cd ~/qt6/src
    wget https://download.qt.io/official_releases/qt/6.5/6.5.1/submodules/qtbase-everywhere-src-6.5.1.tar.xz
    tar xf qtbase-everywhere-src-6.5.1.tar.xz
    ```

3. Build Qt6 for host:

    ```bash
    cd $HOME/qt6/host-build/
    cmake ../src/qtbase-everywhere-src-6.5.1/ -GNinja -DCMAKE_BUILD_TYPE=Release -DQT_BUILD_EXAMPLES=OFF -DQT_BUILD_TESTS=OFF -DCMAKE_INSTALL_PREFIX=$HOME/qt6/host
    cmake --build . --parallel 8
    cmake --install .
    ```

4. Build Qt6 for RPi. Copy necessary folders from Raspberry Pi: Replace `<RPI_IP>` with your Raspberry Pi's IP address (Also user name if it is different than pi)

    ```bash
    cd ~
    rsync -avz --rsync-path="sudo rsync" pi@<RPI_IP>:/usr/include rpi-sysroot/usr
    rsync -avz --rsync-path="sudo rsync" pi@<RPI_IP>:/lib rpi-sysroot
    rsync -avz --rsync-path="sudo rsync" pi@<RPI_IP>:/usr/lib rpi-sysroot/usr 
    rsync -avz --rsync-path="sudo rsync" pi@<RPI_IP>:/opt/vc rpi-sysroot/opt
    ```

5. Create a file named `toolchain.cmake` in `$HOME/qt6`. Replace `<YOUR_USERNAME>` with your username in below file.

    ```cmake
    cmake_minimum_required(VERSION 3.18)
    include_guard(GLOBAL)

    set(CMAKE_SYSTEM_NAME Linux)
    set(CMAKE_SYSTEM_PROCESSOR arm)

    # You should change location of sysroot to your needs.
    set(TARGET_SYSROOT /home/<YOUR_USERNAME>/rpi-sysroot)
    set(TARGET_ARCHITECTURE aarch64-linux-gnu)
    set(CMAKE_SYSROOT ${TARGET_SYSROOT})

    set(ENV{PKG_CONFIG_PATH} $PKG_CONFIG_PATH:${CMAKE_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/pkgconfig)
    set(ENV{PKG_CONFIG_LIBDIR} /usr/lib/pkgconfig:/usr/share/pkgconfig/:${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/pkgconfig:${TARGET_SYSROOT}/usr/lib/pkgconfig)
    set(ENV{PKG_CONFIG_SYSROOT_DIR} ${CMAKE_SYSROOT})

    set(CMAKE_C_COMPILER /opt/cross-pi-gcc/bin/${TARGET_ARCHITECTURE}-gcc)
    set(CMAKE_CXX_COMPILER /opt/cross-pi-gcc/bin/${TARGET_ARCHITECTURE}-g++)

    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -isystem=/usr/include -isystem=/usr/local/include -isystem=/usr/include/${TARGET_ARCHITECTURE}")
    set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS}")

    set(QT_COMPILER_FLAGS "-march=armv8-a")
    set(QT_COMPILER_FLAGS_RELEASE "-O2 -pipe")
    set(QT_LINKER_FLAGS "-Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed -Wl,-rpath-link=${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE} -Wl,-rpath-link=$HOME/qt6/pi/lib")

    set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
    set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
    set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
    set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
    set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
    set(CMAKE_BUILD_RPATH ${TARGET_SYSROOT})

    include(CMakeInitializeConfigs)

    function(cmake_initialize_per_config_variable _PREFIX _DOCSTRING)
    if (_PREFIX MATCHES "CMAKE_(C|CXX|ASM)_FLAGS")
        set(CMAKE_${CMAKE_MATCH_1}_FLAGS_INIT "${QT_COMPILER_FLAGS}")
            
        foreach (config DEBUG RELEASE MINSIZEREL RELWITHDEBINFO)
        if (DEFINED QT_COMPILER_FLAGS_${config})
            set(CMAKE_${CMAKE_MATCH_1}_FLAGS_${config}_INIT "${QT_COMPILER_FLAGS_${config}}")
        endif()
        endforeach()
    endif()


    if (_PREFIX MATCHES "CMAKE_(SHARED|MODULE|EXE)_LINKER_FLAGS")
        foreach (config SHARED MODULE EXE)
        set(CMAKE_${config}_LINKER_FLAGS_INIT "${QT_LINKER_FLAGS}")
        endforeach()
    endif()

    _cmake_initialize_per_config_variable(${ARGV})
    endfunction()

    set(XCB_PATH_VARIABLE ${TARGET_SYSROOT})

    set(GL_INC_DIR ${TARGET_SYSROOT}/usr/include)
    set(GL_LIB_DIR ${TARGET_SYSROOT}:${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/:${TARGET_SYSROOT}/usr:${TARGET_SYSROOT}/usr/lib)

    set(EGL_INCLUDE_DIR ${GL_INC_DIR})
    set(EGL_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libEGL.so)

    set(OPENGL_INCLUDE_DIR ${GL_INC_DIR})
    set(OPENGL_opengl_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libOpenGL.so)

    set(GLESv2_INCLUDE_DIR ${GL_INC_DIR})
    set(GLIB_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libGLESv2.so)

    set(GLESv2_INCLUDE_DIR ${GL_INC_DIR})
    set(GLESv2_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libGLESv2.so)

    set(gbm_INCLUDE_DIR ${GL_INC_DIR})
    set(gbm_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libgbm.so)

    set(Libdrm_INCLUDE_DIR ${GL_INC_DIR})
    set(Libdrm_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libdrm.so)

    set(XCB_XCB_INCLUDE_DIR ${GL_INC_DIR})
    set(XCB_XCB_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libxcb.so)

    list(APPEND CMAKE_LIBRARY_PATH ${CMAKE_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE})
    list(APPEND CMAKE_PREFIX_PATH "/usr/lib/${TARGET_ARCHITECTURE}/cmake")
    ```

6. Fix absolute symbolic links:

    ```bash
    cd ~
    wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
    chmod +x sysroot-relativelinks.py 
    python3 sysroot-relativelinks.py rpi-sysroot
    ```

7. Compile source code for Raspberry Pi:

    ```bash
    cd $HOME/qt6/pi-build
    cmake ../src/qtbase-everywhere-src-6.5.1/ -GNinja -DCMAKE_BUILD_TYPE=Release -DINPUT_opengl=es2 -DQT_BUILD_EXAMPLES=OFF -DQT_BUILD_TESTS=OFF -DQT_HOST_PATH=$HOME/qt6/host -DCMAKE_STAGING_PREFIX=$HOME/qt6/pi -DCMAKE_INSTALL_PREFIX=/usr/local/qt6 -DCMAKE_TOOLCHAIN_FILE=$HOME/qt6/toolchain.cmake -DQT_QMAKE_TARGET_MKSPEC=devices/linux-rasp-pi4-aarch64 -DQT_FEATURE_xcb=ON -DFEATURE_xcb_xlib=ON -DQT_FEATURE_xlib=ON
    cmake --build . --parallel 8
    cmake --install .
    ```

8. Send the binaries to Raspberry Pi: Replace `<RPI_IP>` with your Raspberry Pi's IP address.

    ```bash
    rsync -avz --rsync-path="sudo rsync" $HOME/qt6/pi/* pi@<RPI_IP>:/usr/local/qt6
    ```

## Setting Up Qt Creator

1. Set up **Compilers**

    ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/e98645c4-cf99-45e3-a8b4-ecc0899d6fa0)

2. Set up **Debuggers**

    ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/f75adf17-b8eb-4149-a5fc-cf59978aa3d9)
 
3. Set up **Devices**

    ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/57609ea4-6901-41a8-8264-c6bb7aeac844)

4. Click **Deploy Public Key...** to deploy the key. Create one if not existed.

5. Test the device.

    ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/9883e600-7963-48e3-98fc-dc3f2e651bff)

6. Set up **Qt Versions**.

    ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/6c43b6f0-a256-4d2d-86f6-80bb393602af)

7. Set up **Kits**.

    ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/93e04b07-7cbc-43d6-a17c-53fe6d272de9)

8. On **CMake Configuration** opton, click Change and add follow commands. **You should modify the following commands to your needs.**
dd the following: Replace `<YOUR_USERNAME>` with your actual username.

    ```bash
    -DCMAKE_TOOLCHAIN_FILE:UNINITIALIZED=/home/<YOUR_USERNAME>/qt6/pi/lib/cmake/Qt6/qt.toolchain.cmake
    ```

    ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/d7c4600a-7058-4541-bdfd-ce184e7fd94c)


## Testing HelloWorld

1. In Qt Creator, go to Help > About Plugins and uncheck ClangCodeModel (not needed for Qt Creator 10 or later).

    ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/efb1db08-c5cc-4210-adfe-85507e36d329)

2. Append the following to `CMakeLists.txt` (not needed for Qt Creator 10 or later):

    ```cmake
    install(TARGETS HelloWorld
        RUNTIME DESTINATION ""
        BUNDLE DESTINATION ""
        LIBRARY DESTINATION ""
    )
    ```

3. In the Projects section under Run:
   - Check "Forward to local display" under X11 Forwarding and input `:0`.

   ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/b396954b-fb04-48ae-a3c4-8ae67178513e)

   - Under Environment, add `LD_LIBRARY_PATH` with the value `:/usr/local/qt6/lib/`. Add variable `DISPLAY` with the value `:0` to run on RPI display. Add variable `XAUTHORITY` with the value `/home/pi/.Xauthority`

   ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/059f275c-bfa4-4357-b4b6-82880b5c1054)

4. Run the project to test HelloWorld on Raspberry Pi.

    ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/ee26ad77-f370-433b-8734-89e70c21903c)


## Adding QML Module

1. Download additional source code:

    ```bash
    cd ~/qt6/src
    wget https://download.qt.io/official_releases/qt/6.5/6.5.1/submodules/qtshadertools-everywhere-src-6.5.1.tar.xz
    tar xf qtshadertools-everywhere-src-6.5.1.tar.xz
    wget https://download.qt.io/official_releases/qt/6.5/6.5.1/submodules/qtdeclarative-everywhere-src-6.5.1.tar.xz
    tar xf qtdeclarative-everywhere-src-6.5.1.tar.xz
    ```
    You can check dependencies at ~/qt6/src/qtdeclarative-everywhere-src-6.5.1/dependencies.yaml and ~/qt6/src/qtshadertools-everywhere-src-6.5.1/dependencies.yaml Make sure required modules should be built and installed first.

2. Build the modules for host:

    ```bash
    cd ~/qt6/host-build
    rm -rf *
    $HOME/qt6/host/bin/qt-configure-module ../src/qtshadertools-everywhere-src-6.5.1
    cmake --build . --parallel 8
    cmake --install .
    rm -rf *
    $HOME/qt6/host/bin/qt-configure-module ../src/qtdeclarative-everywhere-src-6.5.1
    cmake --build . --parallel 8
    cmake --install .
    ```

3. Build the modules for Raspberry Pi:

    ```bash
    cd ~/qt6/pi-build
    rm -rf *
    $HOME/qt6/pi/bin/qt-configure-module ../src/qtshadertools-everywhere-src-6.5.1
    cmake --build . --parallel 8
    cmake --install .
    rm -rf *
    $HOME/qt6/pi/bin/qt-configure-module ../src/qtdeclarative-everywhere-src-6.5.1
    cmake --build . --parallel 8
    cmake --install .
    ```

4. Send the binaries to Raspberry Pi: Replace `<RPI_IP>` with your Raspberry Pi's IP address.

    ```bash
    rsync -avz --rsync-path="sudo rsync" $HOME/qt6/pi/* pi@<RPI_IP>:/usr/local/qt6
    ```

## Testing HelloWorldQml

1. Create a new Qt Quick Application project in Qt Creator.
2. Configure the project to use the cross-compiled Qt version for Raspberry Pi.
3. Apply same Run Settings as described in [Testing HelloWorld](#testing-helloworld)
4. Build and run the project on the Raspberry Pi.

    ![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/f67fd349-3537-42f0-8e15-244f138a09d4)


## Adding Serial Bus Module

1. Download additional source code:

    ```bash
    cd ~/qt6/src
    wget https://download.qt.io/official_releases/qt/6.5/6.5.1/submodules/qtserialbus-everywhere-src-6.5.1.tar.xz
    tar xf qtserialbus-everywhere-src-6.5.1.tar.xz
    wget https://download.qt.io/official_releases/qt/6.5/6.5.1/submodules/qtserialport-everywhere-src-6.5.1.tar.xz
    tar xf qtserialport-everywhere-src-6.5.1.tar.xz
    ```
    You can check dependencies at ~/qt6/src/qtserialbus-everywhere-src-6.5.1/dependencies.yaml and ~/qt6/src/qtserialport-everywhere-src-6.5.1/dependencies.yaml Make sure required modules should be built and installed first.

2. Build the modules for host:

    ```bash
    cd ~/qt6/host-build
    rm -rf *
    $HOME/qt6/host/bin/qt-configure-module ../src/qtserialport-everywhere-src-6.5.1
    cmake --build . --parallel 8
    cmake --install .
    rm -rf *
    $HOME/qt6/host/bin/qt-configure-module ../src/qtserialbus-everywhere-src-6.5.1
    cmake --build . --parallel 8
    cmake --install .
    ```

3. Build the modules for Raspberry Pi:

    ```bash
    cd ~/qt6/pi-build
    rm -rf *
    $HOME/qt6/pi/bin/qt-configure-module ../src/qtserialport-everywhere-src-6.5.1
    cmake --build . --parallel 8
    cmake --install .
    rm -rf *
    $HOME/qt6/pi/bin/qt-configure-module ../src/qtserialbus-everywhere-src-6.5.1
    cmake --build . --parallel 8
    cmake --install .
    ```
4. Send the binaries to Raspberry Pi: Replace `<RPI_IP>` with your Raspberry Pi's IP address.

    ```bash
    rsync -avz --rsync-path="sudo rsync" $HOME/qt6/pi/* pi@<RPI_IP>:/usr/local/qt6
    ```

## Troubleshooting

- If you encounter any issues with missing libraries or dependencies, make sure all required packages are installed on both the host system and Raspberry Pi.
- Double-check that all paths in the toolchain.cmake file are correct and match your system configuration.
- Ensure that the Raspberry Pi's IP address is correctly set in all rsync commands.
- If you face problems with X11 forwarding, verify that X11 is properly configured on both the host and Raspberry Pi.

## References
[Cross compilation of Qt6.5.1 for RPI](https://github.com/MuyePan/CrossCompileQtForRpi)