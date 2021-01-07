FROM i386/alpine:3.16 as toolchainbuilder

# Install the build dependencies
RUN apk add --no-cache make m4 which patch gcc musl-dev pkgconfig \
	curl-dev libcurl curl-static musl-utils linux-headers pkgconf-dev \
	zlib-dev zlib-static libarchive-dev libarchive-static nghttp2-static \
	bzip2-static brotli-static lz4-static zstd-static xz-dev \
	openssl-libs-static expat-static acl-static git ninja

RUN git clone https://git.sr.ht/~lattis/muon /muon && \
	cd /muon && \
	./bootstrap.sh build && \
	build/muon setup -D libcurl=enabled build && \
	ninja -C build && \
	build/muon setup -D libcurl=enabled -D samurai=enabled -D static=true build && \
	ninja -C build && \
	build/muon version

FROM madworx/debian-archive:lenny

ENV DEBIAN_FRONTEND=noninteractive

RUN echo "deb http://archive.debian.org/debian lenny main" > /etc/apt/sources.list
RUN echo "deb http://archive.debian.org/debian-security lenny/updates main" >> /etc/apt/sources.list
RUN echo "Acquire::Check-Valid-Until no;" > /etc/apt/apt.conf.d/99no-check-valid-until
RUN apt-get update && apt-get install --yes --force-yes patch unzip libc6-i386 build-essential pkg-config
RUN apt-get install --yes --force-yes checkinstall libbz2-dev libc6-dev libgdbm-dev libncursesw5-dev \
	libreadline-dev libssl-dev libsqlite3-dev

# A binary from https://github.com/moparisthebest/static-curl/releases
COPY curl-i386 /usr/bin/curl
RUN curl --version
# We disable certificate checks
RUN echo "insecure" >> $HOME/.curlrc

# Muon build system from https://sr.ht/~lattis/muon
COPY --from=toolchainbuilder /muon/build/muon /usr/bin/muon
RUN muon version

RUN curl -L -o rizin.zip https://github.com/rizinorg/rizin/archive/dev.zip && \
	unzip rizin.zip && \
	cd rizin-dev/ && \
	muon setup build && \
	muon samu build && \
	muon install

RUN rizin -v && rz-test -v

# TODO: Try to run tests
