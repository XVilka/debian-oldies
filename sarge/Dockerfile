FROM debian/eol:sarge
ARG TCC_COMMIT=4c9e3a5988b4281cba6add9ecc2cf1d6b4650bcf

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
COPY curl-amd64 /usr/bin/curl
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

# R2 doesn't compile because of the old GCC
# Thus we install TinyCC first
RUN curl -L -o tinycc.tar.gz "https://repo.or.cz/tinycc.git/snapshot/${TCC_COMMIT}.tar.gz" && \
	tar -zxvf tinycc.tar.gz && \
	cd tinycc-*/ && \
	./configure && \
	make && \
	make install

# OpenSSL doesn't compile because of the Perl >= 5.10 requirement
# Thus we install custom Perl first
#RUN curl -L -o perl-5.32.0.tar.gz https://www.cpan.org/src/5.0/perl-5.32.0.tar.gz && \
#	tar -zxvf perl-5.32.0.tar.gz && \
#	cd perl-5.32.0/ && \
#	./Configure -des && \
#	make && \
#	make install

# Python 3 doesn't compile because of SSL and zlib errors
# Thus we install custom OpenSSL first
#RUN curl -L -o openssl-1.1.1g.tar.gz https://www.openssl.org/source/openssl-1.1.1g.tar.gz && \
#	tar -zxvf openssl-1.1.1g.tar.gz && \
#	cd openssl-1.1.1g/ && \
#	./config && \
#	make && \
#	make install

#RUN curl -L -o Python-3.6.11.tgz https://www.python.org/ftp/python/3.6.11/Python-3.6.11.tgz && \
#	tar -zxvf Python-3.6.11.tgz && \
#	cd Python-3.6.11/ && \
#	./configure && \
#	make install

RUN curl -L -o radare2.zip https://github.com/radareorg/radare2/archive/master.zip && \
	unzip radare2.zip && \
	cd radare2-master/ && \
	CC=tcc ./configure --with-compiler=tcc && \
	CC=tcc CS_COMMIT_ARCHIVE=1 make && \
	make install

RUN r2 -v && r2r -v

# TODO: Try to run tests


