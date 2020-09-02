FROM i386/alpine:3.12 as tccbuilder
ARG TCC_COMMIT=53d815b8a0364a85b66c3b37884fca087b923267

# Install the build dependencies
RUN apk add --no-cache make m4 which patch curl pkgconfig \
	gcc musl-dev musl-utils linux-headers

ENV CFLAGS="-static"
ENV LDFLAGS="-static"

# TCC doesn't compile because of the old GCC 2.95
# Thus we build TCC statically first, then copy to the container
RUN curl -L -o tinycc.tar.gz "https://repo.or.cz/tinycc.git/snapshot/${TCC_COMMIT}.tar.gz" && \
	tar -zxvf tinycc.tar.gz && \
	cd tinycc-*/ && \
	sed -i "s/#define _VA_LIST_DEFINED/#define __DEFINED_va_list/" include/stdarg.h && \
	./configure --cpu=x86 --enable-static --config-musl --prefix=/usr/local && \
	make && \
	make install

RUN tcc --version
RUN ldd $(which tcc) || true

# Debian Slink has no `unzip` package, so we compile it statically then copy to the container
COPY unzip60.tar.gz unzip60.tar.gz
RUN tar -zxvf unzip60.tar.gz && \
	cd unzip60 && \
	sed -i "s/LFLAGS1 =/LFLAGS1 = -static -no-pie/g" unix/Makefile && \
	sed -i "s/CFLAGS=\"-O3 -Wall -DASM_CRC\"/CFLAGS=\"-O3 -static -no-pie -Wall -DASM_CRC -DNO_LCHMOD\"/g" unix/Makefile && \
	sed -i "s/CFLAGS=\"-O -Wall\"/CFLAGS=\"-O3 -static -no-pie -Wall -DNO_LCHMOD\"/g" unix/Makefile && \
	make -f unix/Makefile linux_noasm && \
	make prefix=/usr/local -f unix/Makefile install

RUN unzip -v
RUN ldd $(which unzip) || true

# Debian Slink can't install `gcc` package and out of the box `make` is too old.
# Thus we compile it statically then copy to the container
RUN curl -L -o make-4.3.tar.gz http://ftp.gnu.org/gnu/make/make-4.3.tar.gz && \
	tar -zxvf make-4.3.tar.gz && \
	cd make-4.3/ && \
	./configure && \
	make && \
	make install

RUN make --version
RUN ldd $(which make) || true

FROM mcandre/docker-debian-32bit:slink
ARG TCC_COMMIT=53d815b8a0364a85b66c3b37884fca087b923267

ENV DEBIAN_FRONTEND=noninteractive

RUN echo "deb http://archive.debian.org/debian slink main" > /etc/apt/sources.list
RUN echo "deb http://archive.debian.org/debian slink-proposed-updates main" >> /etc/apt/source.list
#RUN echo "deb http://archive.debian.org/debian-security slink/updates main" >> /etc/apt/sources.list
RUN echo "Acquire::Check-Valid-Until no;" > /etc/apt/apt.conf
RUN apt-get update && apt-get install --yes --force-yes patch

# A binary from https://github.com/moparisthebest/static-curl/releases
COPY curl-i386 /usr/bin/curl
RUN curl --version
# We disable certificate checks
RUN echo "insecure" >> $HOME/.curlrc

COPY --from=tccbuilder /usr/local /usr/local
# TinyCC doesn't support static builds, so we just copy the libc file
COPY --from=tccbuilder /lib/ld-musl-i386.so.1 /lib/ld-musl-i386.so.1
ENV PATH="/usr/local/bin:${PATH}"
ENV CFLAGS="-std=c11 -D_POSIX_C_SOURCE=200809L"
RUN tcc --version

# R2 doesn't compile because of the old GCC
RUN curl -L -o radare2.zip https://github.com/radareorg/radare2/archive/master.zip
RUN unzip radare2.zip &> /dev/null && \
	cd radare2-master && \
	CC=tcc HOST_CC=tcc ./configure --with-compiler=tcc && \
	CC=tcc HOST_CC=tcc CS_COMMIT_ARCHIVE=1 make && \
	make install

RUN r2 -v && r2r -v

# TODO: Try to run tests

