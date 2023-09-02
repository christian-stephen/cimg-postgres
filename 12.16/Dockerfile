FROM debian:11-slim

LABEL maintainer="Community & Partner Engineering Team <community-partner@circleci.com>"

ENV PG_VER=12.16
ENV PG_MAJOR=12
ENV POSTGRES_HOST_AUTH_METHOD=trust
ENV PATH /usr/lib/postgresql/$PG_MAJOR/bin:$PATH
# We need to maintain the PGDATA directory from the bitnami/postgresql for compatibility reasons.
ENV PGDATA /bitnami/postgresql/data
ENV POSTGRES_DB=circle_test
ENV PGTAP_VERSION 1.2.0
ENV PARTMAN_VERSION 4.7.2

RUN BUILD_DEPS="clang \
        curl \
		dirmngr \
		gnupg \
		git \
		libclang-dev \
		libicu-dev \
		libipc-run-perl \
		libkrb5-dev \
		libldap2-dev \
		liblz4-dev \
		libpam-dev \
		libperl-dev \
		libpython3-dev \
		libreadline-dev \
		libssl-dev \
		libxslt1-dev \
		libxml2-dev \
		llvm \
		llvm-dev \
		pkg-config \
		postgresql-server-dev-all \
		python3-dev \
		tcl-dev \
		uuid-dev" && \
	apt-get update && apt-get install -y --no-install-recommends \
	ca-certificates \
	gosu \
	libicu67 \
	libldap-2.4-2 \
	libreadline8 \
	libxml2 \
	locales \
	make \
	$BUILD_DEPS \
	&& \
	rm -rf /var/lib/apt/lists/* && \
	localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 && \
	curl -sSL "https://ftp.postgresql.org/pub/source/v${PG_VER}/postgresql-${PG_VER}.tar.gz" | tar -xz && \
	cd postgresql-${PG_VER} && \
	./configure \
		--prefix=/usr/lib/postgresql/$PG_MAJOR \
		--enable-integer-datetimes \
		--enable-thread-safety \
		--enable-tap-tests \
		--with-uuid=e2fs \
		--with-gnu-ld \
		--with-pgport=5432 \
		--with-system-tzdata=/usr/share/zoneinfo \
		--with-includes=/usr/local/include \
		--with-libraries=/usr/local/lib \
		--with-krb5 \
		--with-gssapi \
		--with-ldap \
		--with-pam \
		--with-tcl \
		--with-perl \
		--with-python \
		--with-openssl \
		--with-libxml \
		--with-libxslt \
		--with-icu \
		--with-llvm \
		--with-lz4 \
	&& \
	# we can change from world to world-bin in newer releases
	make -j $(nproc) world && \
	make install-world && \
	cd .. && rm -rf postgresql-${PG_VER} \
	&& \
	git clone --depth 1 https://github.com/citusdata/pg_cron.git /pg_cron && \
	cd /pg_cron && make && PATH=$PATH make install && \
    cd .. && rm -rf /pg_cron \
	&& \
	curl -sSL "https://github.com/pgpartman/pg_partman/archive/v${PARTMAN_VERSION}.tar.gz" | tar -xz && \
	cd pg_partman-${PARTMAN_VERSION} && make NO_BGW=1 install && \
	cd .. && rm -rf pg_partman-${PARTMAN_VERSION} && \
	apt-get purge -y --auto-remove $BUILD_DEPS

COPY pg_cron.sh /docker-entrypoint-initdb.d/
COPY custom-postgresql.conf /etc/postgresql/custom-postgresql.conf
COPY docker-entrypoint.sh /usr/local/bin/

# Modify the postgres user to match the one used in the bitnami/postgresql image.
RUN usermod -u 1001 -g 0 postgres && \
    chmod +x /usr/local/bin/docker-entrypoint.sh /docker-entrypoint-initdb.d/pg_cron.sh && \
	mkdir -p /bitnami/postgresql && \
	# Symlink for backwards compatibility.
	ln -s usr/local/bin/docker-entrypoint.sh / && \
	chown -R postgres:postgres /bitnami/postgresql && \
	chown -R postgres:postgres /etc/postgresql && \
	chown -R postgres:postgres /usr/local/bin/docker-entrypoint.sh

USER 1001
ENTRYPOINT ["docker-entrypoint.sh"]
STOPSIGNAL SIGINT
EXPOSE 5432
CMD ["postgres"]
