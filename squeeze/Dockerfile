FROM debian:squeeze

ENV DEBIAN_FRONTEND=noninteractive

RUN echo "deb http://archive.debian.org/debian squeeze main" > /etc/apt/sources.list
RUN echo "deb http://archive.debian.org/debian squeeze-lts main" >> /etc/apt/sources.list
RUN echo "deb http://archive.debian.org/debian-security squeeze/updates main" >> /etc/apt/sources.list
RUN echo "Acquire::Check-Valid-Until no;" > /etc/apt/apt.conf.d/99no-check-valid-until
RUN apt-get update && apt-get install --yes --force-yes patch unzip libc6-i386 build-essential pkg-config
RUN apt-get install --yes --force-yes checkinstall libbz2-dev libc6-dev libgdbm-dev libncursesw5-dev \
	libreadline-dev libssl-dev libsqlite3-dev

# A binary from https://github.com/moparisthebest/static-curl/releases
COPY curl-amd64 /usr/bin/curl
RUN curl --version
# We disable certificate checks
RUN echo "insecure" >> $HOME/.curlrc

RUN curl -L -o Python-3.6.11.tgz https://www.python.org/ftp/python/3.6.11/Python-3.6.11.tgz && \
	tar -zxvf Python-3.6.11.tgz && \
	cd Python-3.6.11/ && \
	./configure && \
	make install

RUN curl -L -o rizin.zip https://github.com/rizinorg/rizin/archive/dev.zip && \
	unzip rizin.zip && \
	cd rizin-dev/ && \
	./configure && \
	CS_COMMIT_ARCHIVE=1 make && \
	make install

RUN rizin -v && rz-test -v

# TODO: Try to run tests
