FROM ubuntu:16.04 AS BUILDSTEP
MAINTAINER Sergey Dolin <sergey@s4y.solutions>

# Install all necessary for building dependecnies, read more about
# depencensies in readme section in the root of the repo.
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y wget vim psmisc git libev-dev libmpdec-dev  libmysqlclient-dev libssl-dev \
        build-essential autoconf libtool python\
    && rm -rf /var/lib/apt/lists/* 

#install jansson
RUN git clone https://github.com/akheron/jansson \
    && cd ./jansson \
    && autoreconf -i \
    && ./configure \
    && make \
    && make install \
    && cd ../

#install kafka
RUN git clone https://github.com/edenhill/librdkafka.git \
    && cd librdkafka \
    && ./configure \
    && make \
    && make install \
    && cd / \
    && rm -rf librdkafka

#install curl    
RUN wget --no-check-certificate https://codeload.github.com/curl/curl/tar.gz/curl-7_45_0 -O curl-7.45.0.tar.gz \
    && tar xzvf curl-7.45.0.tar.gz \
    && rm -rf curl-7.45.0.tar.gz \
    && mv curl-* curl \
    && cd ./curl \
    && ./buildconf \
    && ./configure --prefix=/usr/local --disable-ldap --disable-ldaps \
    && make \
    && make install \
    && cd ../    

RUN mkdir viabtc_exchange_server

# Copy all necessary files and directories for build process, basically we
# recreate the root repository directory, but with tham we could ensure that
# nothing
COPY depends    viabtc_exchange_server/depends
COPY network    viabtc_exchange_server/network
COPY utils      viabtc_exchange_server/utils

COPY alertcenter    viabtc_exchange_server/alertcenter
COPY matchengine    viabtc_exchange_server/matchengine
COPY marketprice    viabtc_exchange_server/marketprice
COPY readhistory    viabtc_exchange_server/readhistory
COPY accesshttp     viabtc_exchange_server/accesshttp
COPY accessws       viabtc_exchange_server/accessws

COPY makefile.inc                                   viabtc_exchange_server/makefile.inc

# Do the actial build of the microservices.
RUN cd viabtc_exchange_server/depends/hiredis \
    && make \
    && make install \
    && cd ../../

RUN cd viabtc_exchange_server/network \
    && make

RUN cd viabtc_exchange_server/utils \
    && make

RUN cd viabtc_exchange_server/alertcenter \
    && make

RUN cd viabtc_exchange_server/matchengine \
    && make

RUN cd viabtc_exchange_server/marketprice \
    && make

RUN cd viabtc_exchange_server/readhistory \
    && make

RUN cd viabtc_exchange_server/accesshttp \
    && make

RUN cd viabtc_exchange_server/accessws \
    && make

# Multi-stage builds are a new feature requiring Docker 17.05 or higher on the
# daemon and client. Multistage builds are useful to anyone who has struggled
# to optimize Dockerfiles while keeping them easy to read and maintain.
# With this we ensure that previous build layers are not stored and we grab
# only what we need.
FROM ubuntu:16.04

RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get install -y \
            liblz4-dev \
			libldap-2.4 \
			libgss3 \
			librtmp1 \
			libmysqlclient20 \
			jq \
			netcat
RUN rm -rf /var/lib/apt/lists/*

RUN mkdir /viabtc_exchange_server
# keep .exe for better grep of ps output
COPY --from=BUILDSTEP   /viabtc_exchange_server/alertcenter/alertcenter \
                        /viabtc_exchange_server/alertcenter.exe
COPY docker/exchange/configs/alertcenter.json /viabtc_exchange_server

COPY --from=BUILDSTEP   /viabtc_exchange_server/matchengine/matchengine \
                        /viabtc_exchange_server/matchengine.exe
COPY docker/exchange/configs/matchengine.json /viabtc_exchange_server

COPY --from=BUILDSTEP   /viabtc_exchange_server/marketprice/marketprice \
                        /viabtc_exchange_server/marketprice.exe
COPY docker/exchange/configs/marketprice.json /viabtc_exchange_server

COPY --from=BUILDSTEP   /viabtc_exchange_server/readhistory/readhistory \
                        /viabtc_exchange_server/readhistory.exe
COPY docker/exchange/configs/readhistory.json /viabtc_exchange_server

COPY --from=BUILDSTEP   /viabtc_exchange_server/accesshttp/accesshttp \
                        /viabtc_exchange_server/accesshttp.exe
COPY docker/exchange/configs/accesshttp.json /viabtc_exchange_server

COPY --from=BUILDSTEP   /viabtc_exchange_server/accessws/accessws \
                        /viabtc_exchange_server/accessws.exe
COPY docker/exchange/configs/accessws.json /viabtc_exchange_server

COPY docker/exchange/entrypoint.sh /docker-entrypoint.sh
RUN ["chmod", "+x", "/docker-entrypoint.sh"]

ENTRYPOINT [ "/docker-entrypoint.sh" ]
