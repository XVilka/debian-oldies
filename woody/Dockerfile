FROM i386/alpine:3.12 as tccbuilder
ARG TCC_COMMIT=696b765437b4a9213e3883ff416034daaebc891c

# Install the build dependencies
RUN apk add --no-cache make m4 which patch curl pkgconfig \
	gcc musl-dev musl-utils linux-headers

ENV CFLAGS="-static"

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
RUN ldd $(which tcc)

FROM debian/eol:woody
ARG TCC_COMMIT=696b765437b4a9213e3883ff416034daaebc891c

ENV DEBIAN_FRONTEND=noninteractive

RUN echo "deb http://archive.debian.org/debian woody main" > /etc/apt/sources.list
RUN echo "deb http://archive.debian.org/debian-security woody/updates main" >> /etc/apt/sources.list
RUN echo "Acquire::Check-Valid-Until no;" > /etc/apt/apt.conf.d/99no-check-valid-until
RUN apt-get update && apt-get install --yes --force-yes patch unzip libc6 build-essential pkg-config

# A binary from https://github.com/moparisthebest/static-curl/releases
COPY curl-i386 /usr/bin/curl
RUN curl --version
# We disable certificate checks
RUN echo "insecure" >> $HOME/.curlrc

# R2 doesn't compile because of the old GNU Make
# Thus we install custom make first
RUN curl -L -o make-4.3.tar.gz http://ftp.gnu.org/gnu/make/make-4.3.tar.gz && \
	tar -zxvf make-4.3.tar.gz && \
	cd make-4.3/ && \
	./configure && \
	make && \
	make install

COPY --from=tccbuilder /usr/local /usr/local
# TinyCC doesn't support static builds, so we just copy the libc file
COPY --from=tccbuilder /lib/ld-musl-i386.so.1 /lib/ld-musl-i386.so.1
ENV PATH="/usr/local/bin:${PATH}"
RUN tcc --version

# R2 doesn't compile because of the old GCC
# Thus we install TinyCC first
# Old sed doesn't know "-i" option, thus using the intermediate file
# FIXME: For some reason produced `tcc` gives 'Segmentation fault' when running
#RUN curl -L -o tinycc.tar.gz "https://repo.or.cz/tinycc.git/snapshot/${TCC_COMMIT}.tar.gz" && \
#	tar -zxvf tinycc.tar.gz && \
#	cd tinycc-*/ && \
#	sed "115s/.*/#elif 1/" lib/bcheck.c > lib/bcheck.c.new && \
#	sed "132s/.*/#define HAVE_CTYPE		(0)/" lib/bcheck.c.new > lib/bcheck.c.new2 && \
#	rm -f lib/bcheck.c.new && \
#	mv -f lib/bcheck.c.new2 lib/bcheck.c && \
#	./configure --cpu=x86 --cc=tcc && \
#	make && \
#	# Remove the Alpine-built TinyCC version first
#	rm -rf /usr/local/* && \
#	make install
#RUN tcc --version

RUN curl -L -o radare2.zip https://github.com/radareorg/radare2/archive/master.zip && \
	unzip radare2.zip && \
	cd radare2-master/ && \
	CC=tcc HOST_CC=tcc ./configure --with-compiler=tcc && \
	CC=tcc HOST_CC=tcc CS_COMMIT_ARCHIVE=1 make && \
	make install

RUN r2 -v && r2r -v

# TODO: Try to run tests

