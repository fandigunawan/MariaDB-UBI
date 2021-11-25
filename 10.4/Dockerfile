ARG BASE_IMAGE=redhat/ubi8
ARG BASE_TAG=8.5

FROM ${BASE_IMAGE}:${BASE_TAG}

ENV MARIADB_MAJOR 10.4
ENV MARIADB_VERSION 10.4.22

# copy dependencies, GPG keys, and scripts
COPY mariadb-server.rpm mariadb-client.rpm mariadb-common.rpm mariadb-shared.rpm galera.rpm boost.rpm signatures/RPM-GPG-KEY-MariaDB signatures/RPM-GPG-KEY-CentOS-Official /
COPY scripts/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
COPY scripts/healthcheck.sh /usr/local/bin/healthcheck.sh

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
# import GPG keys, update, install dependencies, and clean
# remove rpms/keys and allow exec permissions on entrypoint
RUN groupadd -g 27 -r mysql && \
    useradd -u 27 -r -g mysql mysql && \
    rpm --import RPM-GPG-KEY-MariaDB && \
    rpm --import RPM-GPG-KEY-CentOS-Official && \
    dnf update -y && \
    dnf install -y mariadb-server.rpm mariadb-client.rpm mariadb-common.rpm mariadb-shared.rpm galera.rpm boost.rpm --setopt=tsflags=nodocs && \
    dnf clean all && \
    rm -rf /var/cache/dnf && \
    rm -f mariadb-server.rpm mariadb-client.rpm mariadb-common.rpm mariadb-shared.rpm galera.rpm boost.rpm RPM-GPG-KEY-MariaDB RPM-GPG-KEY-CentOS-Official && \
    chmod +x /usr/local/bin/docker-entrypoint.sh && \
    chmod +x /usr/local/bin/healthcheck.sh && \
    # Default installation has preconfigured credentials and sample databases.
    # We're going to delete all of it and allow the entrypoint to recreate things securely.
    rm -rf /var/lib/mysql && \
	mkdir -p /var/lib/mysql /var/run/mysqld && \
	chown -R mysql:mysql /var/lib/mysql /var/run/mysqld && \
    mkdir /docker-entrypoint-initdb.d && \
    chown mysql:mysql /docker-entrypoint-initdb.d

COPY config/my.cnf /etc/my.cnf
COPY config/server.cnf /etc/my.cnf.d/server.cnf

USER mysql

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
HEALTHCHECK CMD /usr/local/bin/healthcheck.sh
EXPOSE 3306

CMD ["mysqld"]
