FROM ubuntu:20.04

ARG repository="deb https://repo.clickhouse.tech/deb/stable/ main/"
ARG version=21.3.3.*
ARG gosu_ver=1.10

# user/group precreated explicitly with fixed uid/gid on purpose.
# It is especially important for rootless containers: in that case entrypoint
# can't do chown and owners of mounted volumes should be configured externally.
# We do that in advance at the begining of Dockerfile before any packages will be
# installed to prevent picking those uid / gid by some unrelated software.
# The same uid / gid (101) is used both for alpine and ubuntu.

RUN groupadd -r clickhouse --gid=101 \
    && useradd -r -g clickhouse --uid=101 --home-dir=/var/lib/clickhouse --shell=/bin/bash clickhouse \
    && apt-get update \
    && apt-get install --yes --no-install-recommends \
        apt-transport-https \
        ca-certificates \
        dirmngr \
        gnupg \
    && mkdir -p /etc/apt/sources.list.d \
    && apt-key adv --keyserver keyserver.ubuntu.com --recv E0C56BD4 \
    && echo $repository > /etc/apt/sources.list.d/clickhouse.list \
    && apt-get update \
    && env DEBIAN_FRONTEND=noninteractive \
        apt-get --yes -o "Dpkg::Options::=--force-confdef" -o "Dpkg::Options::=--force-confold" upgrade \
    && env DEBIAN_FRONTEND=noninteractive \
        apt-get install --allow-unauthenticated --yes --no-install-recommends \
            clickhouse-common-static=$version \
            clickhouse-client=$version \
            clickhouse-server=$version \
            locales \
            wget \
    && rm -rf \
        /var/lib/apt/lists/* \
        /var/cache/debconf \
        /tmp/* \
    && apt-get clean \
    && mkdir -p /var/lib/clickhouse /var/log/clickhouse-server /etc/clickhouse-server /etc/clickhouse-client \
    && chmod ugo+Xrw -R /var/lib/clickhouse /var/log/clickhouse-server /etc/clickhouse-server /etc/clickhouse-client

# we need to allow "others" access to clickhouse folder, because docker container
# can be started with arbitrary uid (openshift usecase)

ADD https://github.com/tianon/gosu/releases/download/$gosu_ver/gosu-amd64 /bin/gosu

RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
ENV TZ UTC

RUN mkdir /docker-entrypoint-initdb.d

# User tier to point to differnt .xml files for prod/stage/dev
ARG tier
COPY docker_related_config_${tier}.xml /etc/clickhouse-server/config.d/docker_related_config.xml
COPY window_function.xml /etc/clickhouse-server/users.d/window_function.xml

#Define custom dictionaries
#COPY master_users_dictionary_${tier}.xml /etc/clickhouse-server/master_users_dictionary.xml
COPY merch_product_tags_dictionary_${tier}.xml /etc/clickhouse-server/merch_product_tags_dictionary.xml
COPY commerce_merchant_detected_country_dictionary_${tier}.xml /etc/clickhouse-server/commerce_merchant_detected_country_dictionary.xml
#COPY contest_dictionary_${tier}.xml /etc/clickhouse-server/contest_dictionary.xml

COPY entrypoint.sh /entrypoint.sh

RUN chmod +x \
    /entrypoint.sh \
    /bin/gosu

EXPOSE 9000 8123 9009
VOLUME /var/lib/clickhouse

ENV CLICKHOUSE_CONFIG /etc/clickhouse-server/config.xml

ENTRYPOINT ["/entrypoint.sh"]

