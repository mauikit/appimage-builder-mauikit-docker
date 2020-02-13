FROM appimagecrafters/appimage-builder:ubuntu-18.04-arm64

RUN apt-get update
RUN apt-get install curl -y

# setup kde neon repositories
RUN echo "deb http://archive.neon.kde.org/user bionic main" > /etc/apt/sources.list.d/neon.list
RUN curl https://origin.archive.neon.kde.org/public.key -sS | apt-key add -
RUN apt-get update

# install toolchain
RUN apt-get install -y g++ cmake git gettext checkinstall

# install mauikit dependencies
RUN apt-get install -y libkf5configwidgets-dev libkf5kio-dev libkf5service-dev libkf5config-dev \
  libkf5notifications-dev libkf5i18n-dev extra-cmake-modules libqt5svg5-dev qtquickcontrols2-5-dev \
  qtdeclarative5-dev qtwebengine5-dev libkf5syntaxhighlighting-dev

# Build mauikit
RUN git clone https://invent.kde.org/kde/mauikit.git /tmp/mauikit --depth=1 --branch master
WORKDIR /tmp/mauikit
RUN cmake . -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release
RUN make -j$(nproc)

# Install mauikit
RUN checkinstall --pkgname mauikit --pkgversion 0.1.0 -y --backup=no

WORKDIR /
RUN rm -rf /tmp/mauikit
RUN apt-get autoclean