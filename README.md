### Sets up pdftron and php5.6-cli using the following the following `Dockerfile`

```bash
FROM debian:jessie-slim
ENV LANG en_US.UTF-8
LABEL name=pdftron-php-5.6 version=2.0.1 \
      maintainer="swarajgiri@gmail.com"

# Install dependencies
RUN DEBIAN_FRONTEND=noninteractive apt-get update -yqq -o Acquire::CompressionTypes::Order::=gz && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y apt-utils locales wget git cmake \
    zip unzip swig2.0

# Set the locale
RUN localedef -i en_US -f UTF-8 en_US.UTF-8

# Install php
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 14AA40EC0831756756D7F66C4F4EA0AAE5267A6C \
 && echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu trusty main" >> /etc/apt/sources.list \
 && DEBIAN_FRONTEND=noninteractive apt-get update -qq \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y php5.6 php5.6-dev php5.6-zip \
 php5.6-json php5.6-xml php5.6-mbstring \
 && apt-get clean

# Build pdftron wrappers
RUN mkdir wrappers_build && cd wrappers_build \
 && git clone https://github.com/PDFTron/PDFNetWrappers \
 && cd PDFNetWrappers/PDFNetC \
 && wget http://www.pdftron.com/downloads/PDFNetC64.tar.gz \
 && tar xzvf PDFNetC64.tar.gz \
 && mv PDFNetC64/Headers/ . \
 && mv PDFNetC64/Lib/ . \
 && cd .. \
 && mkdir Build \
 && cd Build \
 && cmake -D BUILD_PDFNetPHP=ON .. \
 && make && make install \
 && echo "extension=/usr/lib/php/20131226/PDFNetPHP.so" >> /etc/php/5.6/cli/php.ini

# Set current directory
WORKDIR wrappers_build/PDFNetWrappers/Samples

# Execute all php tests
ENTRYPOINT ["/bin/bash"]
CMD ["runall_php.sh"]

```
