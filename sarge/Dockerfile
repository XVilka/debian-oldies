FROM debian/eol:sarge
ARG TCC_COMMIT=727e24cb0ac7b6f73658c5d9b88e35723f596fba

ENV DEBIAN_FRONTEND=noninteractive

RUN echo "deb http://archive.debian.org/debian sarge main" > /etc/apt/sources.list
RUN echo "deb http://archive.debian.org/debian-security sarge/updates main" >> /etc/apt/sources.list
RUN echo "Acquire::Check-Valid-Until no;" > /etc/apt/apt.conf.d/99no-check-valid-until
RUN apt-get update && apt-get install --yes --force-yes patch unzip libc6 build-essential pkg-config
# Perl build dependencies
#RUN apt-get install --yes --force-yes libmodule-install-perl libmodule-install-readmefrompod-perl
# Python build dependencies
RUN apt-get install --yes --force-yes libbz2-dev libc6-dev libgdbm-dev libncursesw5-dev \
	libreadline5-dev libssl-dev libsqlite3-dev

# A binary from https://github.com/moparisthebest/static-curl/releases
COPY curl-i386 /usr/bin/curl
RUN curl --version
# We disable certificate checks
RUN echo "insecure" >> $HOME/.curlrc

# Rizin doesn't compile because of the old GNU Make
# Thus we install custom make first
RUN curl -L -o make-4.3.tar.gz http://ftp.gnu.org/gnu/make/make-4.3.tar.gz && \
	tar -zxvf make-4.3.tar.gz && \
	cd make-4.3/ && \
	./configure && \
	make && \
	make install

# Rizin doesn't compile because of the old GCC
# Thus we install TinyCC first
RUN curl -L -o tinycc.tar.gz "https://repo.or.cz/tinycc.git/snapshot/${TCC_COMMIT}.tar.gz" && \
	tar -zxvf tinycc.tar.gz && \
	cd tinycc-*/ && \
	sed -i "115s/.*/#elif 1/" lib/bcheck.c && \
	./configure --cpu=x86 && \
	make && \
	make install

RUN curl -L -o rizin.zip https://github.com/rizinorg/rizin/archive/dev.zip && \
	unzip rizin.zip && \
	cd rizin-dev/ && \
	CC=tcc HOST_CC=tcc ./configure --with-compiler=tcc && \
	CC=tcc HOST_CC=tcc CS_COMMIT_ARCHIVE=1 make -j1 && \
	make install

RUN rizin -v && rz-test -v

# TODO: Try to run tests


