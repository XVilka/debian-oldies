FROM madworx/debian-archive:lenny

ENV DEBIAN_FRONTEND=noninteractive

RUN echo "deb http://archive.debian.org/debian lenny main" > /etc/apt/sources.list
RUN echo "deb http://archive.debian.org/debian-security lenny/updates main" >> /etc/apt/sources.list
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

RUN curl -L -o radare2.zip https://github.com/radareorg/radare2/archive/master.zip && \
	unzip radare2.zip && \
	cd radare2-master/ && \
	./configure && \
	CS_COMMIT_ARCHIVE=1 make && \
	make install

RUN r2 -v && r2r -v

# TODO: Try to run tests
