FROM java:8-jre

ARG GRAYLOG_VERSION

ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64/jre
ENV GRAYLOG_SERVER_JAVA_OPTS "-Xms1g -Xmx2g -XX:NewRatio=1 -XX:MaxMetaspaceSize=256m -server -XX:+ResizeTLAB -XX:+UseConcMarkSweepGC -XX:+CMSConcurrentMTEnabled -XX:+CMSClassUnloadingEnabled -XX:+UseParNewGC -XX:-OmitStackTraceInFastThrow"
ENV GOSU_VERSION 1.7

RUN set -x \
  && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
  && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
  && export GNUPGHOME="$(mktemp -d)" \
  && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
  && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
  && rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
  && chmod +x /usr/local/bin/gosu \
  && gosu nobody true


RUN set -x \
  && apt-get update \
  && apt-get dist-upgrade -y \
  && rm -rf /var/lib/apt/lists/*

RUN set -x \
  && addgroup --gid 1100 graylog \
  && adduser --disabled-password --disabled-login --gecos '' --uid 1100 --gid 1100 graylog \
  && mkdir /usr/share/graylog \
  && wget -O /usr/share/graylog.tgz "https://packages.graylog2.org/releases/graylog/graylog-${GRAYLOG_VERSION}.tgz" \
  && tar xfz /usr/share/graylog.tgz --strip-components=1 -C /usr/share/graylog \
  && chown -R graylog:graylog /usr/share/graylog \
  && rm /usr/share/graylog.tgz \
  && setcap 'cap_net_bind_service=+ep' $JAVA_HOME/bin/java


ENV PATH /usr/share/graylog/bin:$PATH
WORKDIR /usr/share/graylog

RUN set -ex \
  && for path in \
    ./data/journal \
    ./data/log \
    ./data/config \
  ; do \
    mkdir -p "$path"; \
  done

COPY config/* ./data/config/

RUN chown -R graylog:graylog ./data

VOLUME /usr/share/graylog/data

COPY docker-entrypoint.sh /

EXPOSE 9000 12900 12201 12201/udp
USER root
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["graylog"]
