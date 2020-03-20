FROM appimagecrafters/appimage-builder

RUN apt-get update
RUN apt-get install curl -y


# add foreign architecture for cross-compiling
RUN dpkg --add-architecture arm64

# setup kde neon repositories
RUN echo "deb [arch=amd64] http://archive.ubuntu.com/ubuntu bionic main multiverse universe" > /etc/apt/sources.list && \
    echo "deb [arch=amd64] http://archive.ubuntu.com/ubuntu bionic-updates main multiverse universe" >> /etc/apt/sources.list && \
    echo "deb [arch=amd64] http://archive.ubuntu.com/ubuntu bionic-security main multiverse universe" >> /etc/apt/sources.list

RUN apt-get update
RUN apt-get dist-upgrade -y

# install toolchain
RUN apt-get -y -f --no-install-recommends install crossbuild-essential-arm64 cmake git gettext checkinstall \
    pkg-config-aarch64-linux-gnu qemu-user-static qt5-qmake qtbase5-dev-tools

ARG BUILD_ROOT=/tmp/buildroot
WORKDIR $BUILD_ROOT

ADD apt.conf etc/apt/
ADD sources.list etc/apt/

RUN mkdir -p \
        etc/apt/trusted.gpg.d etc/apt/trusted.gpg.d \
        var/lib/apt/lists/partial var/lib/dpkg \
        var/cache/apt/archives/partial var/log/apt; \

    touch var/lib/dpkg/status etc/apt/trusted.gpg.d/neon.gpg; \
    cp -r /etc/apt/trusted.gpg.d etc/apt; \

    # add kde neon repo key
    curl https://origin.archive.neon.kde.org/public.key -sS | apt-key --keyring $BUILD_ROOT/etc/apt/trusted.gpg.d/neon.gpg add -; \
    apt-get -c etc/apt/apt.conf update; \

    # install mauikit dependencies
    apt-get -c etc/apt/apt.conf -y --no-install-recommends --allow-remove-essential --download-only install \
        libc-dev:arm64 libicu-dev:arm64 \
        libkf5configwidgets-dev:arm64 libkf5kio-dev:arm64 \
        libkf5service-dev:arm64 libkf5config-dev:arm64 libkf5notifications-dev:arm64 libkf5i18n-dev:arm64 \
        extra-cmake-modules:arm64 libqt5svg5-dev:arm64 qtquickcontrols2-5-dev:arm64 qtdeclarative5-dev:arm64 \
        qtwebengine5-dev:arm64 libkf5syntaxhighlighting-dev:arm64 libqt5websockets5-dev:arm64 \
        qtmultimedia5-dev:arm64 libqt5webview5-dev:arm64 libqt5x11extras5-dev:arm64 \
        libqt5webkit5-dev:arm64 libqt5webchannel5-dev:arm64 libqt5virtualkeyboard5-dev:arm64 \
        libqt5sql5-sqlite:arm64 libkf5auth-dev:arm64 libkf5coreaddons-dev:arm64 libkf5bookmarks-dev:arm64 \
        libkf5widgetsaddons-dev:arm64 libkf5completion-dev:arm64 libkf5codecs-dev:arm64 \
        libtag1-dev:arm64 libkf5contacts-dev:arm64 libkf5people-dev:arm64 ; \

    # extract downloaded files
    for DEB_FILE in var/cache/apt/archives/*.deb; do \
        echo "Extracting: $DEB_FILE"; \
        dpkg-deb -x $DEB_FILE $BUILD_ROOT; \
    done ;\

    # Copy libs and headers to the file system root
    cp -rn lib/* /lib ;\
    cp -rn usr/lib/* /usr/lib ;\
    cp -rn usr/include/* /usr/include/ ;\
    cp -rn usr/share/ECM /usr/share ;\
    cp -rn usr/bin/* /usr/bin ;\

    # cleanup
    rm -rf $BUILD_ROOT

# Generated from `dpkg-architecture -a arm64`
ENV DEB_BUILD_ARCH=amd64 \
    DEB_BUILD_ARCH_BITS=64 \
    DEB_BUILD_ARCH_CPU=amd64 \
    DEB_BUILD_ARCH_ENDIAN=little \
    DEB_BUILD_ARCH_OS=linux \
    DEB_BUILD_GNU_CPU=x86_64 \
    DEB_BUILD_GNU_SYSTEM=linux-gnu \
    DEB_BUILD_GNU_TYPE=x86_64-linux-gnu \
    DEB_BUILD_MULTIARCH=x86_64-linux-gnu \
    DEB_HOST_ARCH=arm64 \
    DEB_HOST_ARCH_BITS=64 \
    DEB_HOST_ARCH_CPU=arm64 \
    DEB_HOST_ARCH_ENDIAN=little \
    DEB_HOST_ARCH_OS=linux \
    DEB_HOST_GNU_CPU=aarch64 \
    DEB_HOST_GNU_SYSTEM=linux-gnu \
    DEB_HOST_GNU_TYPE=aarch64-linux-gnu \
    DEB_HOST_MULTIARCH=aarch64-linux-gnu \
    DEB_TARGET_ARCH=arm64 \
    DEB_TARGET_ARCH_BITS=64 \
    DEB_TARGET_ARCH_CPU=arm64 \
    DEB_TARGET_ARCH_ENDIAN=little \
    DEB_TARGET_ARCH_OS=linux \
    DEB_TARGET_GNU_CPU=aarch64 \
    DEB_TARGET_GNU_SYSTEM=linux-gnu \
    DEB_TARGET_GNU_TYPE=aarch64-linux-gnu \
    DEB_TARGET_MULTIARCH=aarch64-linux-gnu \
    PKG_CONFIG_LIBDIR=$BUILD_ROOT/usr/lib/aarch64-linux-gnu/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig \
    CC=aarch64-linux-gnu-gcc \
    CXX=aarch64-linux-gnu-g++

ADD arm64-toolchain.cmake /

# Build mauikit
RUN git clone https://invent.kde.org/kde/mauikit.git /tmp/mauikit --depth=1 --branch master
WORKDIR /tmp/mauikit

RUN cmake . -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE='/arm64-toolchain.cmake' ;\
    make -j$(nproc) ;\

    # Install mauikit
    checkinstall --pkgname=mauikit --pkgversion=0.1.0 --pkgarch=arm64 -y --backup=no ;\
    rm -rf /tmp/mauikit

RUN apt-get autoremove checkinstall ;\
    apt-get autoclean
