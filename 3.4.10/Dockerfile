FROM ubuntu:16.04

ENV ZK_USER=zookeeper
ENV ZK_DATA_DIR=/var/lib/zookeeper/data
ENV ZK_DATA_LOG_DIR=/var/lib/zookeeper/log
ENV ZK_LOG_DIR=/var/log/zookeeper
ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
ENV ZK_REPLICAS=3
ENV ZK_LOG_LEVEL=INFO
ENV ZK_CONF_DIR=/opt/zookeeper/conf
ENV ZK_CLIENT_PORT=2181
ENV ZK_SERVER_PORT=2888
ENV ZK_ELECTION_PORT=3888
ENV ZK_INIT_LIMIT=10
ENV ZK_SYNC_LIMIT=5
ENV ZK_HEAP_SIZE=2G
ENV ZK_MAX_CLIENT_CNXNS=60
ENV ZK_TICK_TIME=2000
ENV ZK_MIN_SESSION_TIMEOUT=4000
ENV ZK_MAX_SESSION_TIMEOUT=40000
ENV ZK_SNAP_RETAIN_COUNT=3
ENV ZK_PURGE_INTERVAL=0
ENV ID_FILE="$ZK_DATA_DIR/myid"
ENV ZK_CONFIG_FILE="$ZK_CONF_DIR/zoo.cfg"
ENV LOGGER_PROPS_FILE="$ZK_CONF_DIR/log4j.properties"
ENV JAVA_ENV_FILE="$ZK_CONF_DIR/java.env"

ARG GPG_KEY=C823E3E5B12AF29C67F81976F5CECB3CB5E9BD2D
ARG ZK_DIST=zookeeper-3.4.10

RUN set -x \
    && apt-get update \
    && apt-get install -y openjdk-8-jre-headless wget netcat-openbsd \
    && wget -q "http://www.apache.org/dist/zookeeper/$ZK_DIST/$ZK_DIST.tar.gz" \
    && wget -q "http://www.apache.org/dist/zookeeper/$ZK_DIST/$ZK_DIST.tar.gz.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-key "$GPG_KEY" \
    && gpg --batch --verify "$ZK_DIST.tar.gz.asc" "$ZK_DIST.tar.gz" \
    && tar -xzf "$ZK_DIST.tar.gz" -C /opt \
    && rm -r "$GNUPGHOME" "$ZK_DIST.tar.gz" "$ZK_DIST.tar.gz.asc" \
    && ln -s /opt/$ZK_DIST /opt/zookeeper \
    && rm -rf /opt/zookeeper/CHANGES.txt \
    /opt/zookeeper/README.txt \
    /opt/zookeeper/NOTICE.txt \
    /opt/zookeeper/CHANGES.txt \
    /opt/zookeeper/README_packaging.txt \
    /opt/zookeeper/build.xml \
    /opt/zookeeper/config \
    /opt/zookeeper/contrib \
    /opt/zookeeper/dist-maven \
    /opt/zookeeper/docs \
    /opt/zookeeper/ivy.xml \
    /opt/zookeeper/ivysettings.xml \
    /opt/zookeeper/recipes \
    /opt/zookeeper/src \
    /opt/zookeeper/$ZK_DIST.jar.asc \
    /opt/zookeeper/$ZK_DIST.jar.md5 \
    /opt/zookeeper/$ZK_DIST.jar.sha1 \
    && apt-get autoremove -y wget \
    && rm -rf /var/lib/apt/lists/*

VOLUME /var/lib/zookeeper
EXPOSE 2181

# Copy configuration generator script to bin
COPY bootstrap.sh zkOk.sh zkMetrics.sh /opt/zookeeper/bin/

# Create a user for the zookeeper process and configure file system ownership
# for necessary directories and symlink the distribution as a user executable
RUN set -x \
    && useradd $ZK_USER \
    && [ `id -u $ZK_USER` -eq 1000 ] \
    && [ `id -g $ZK_USER` -eq 1000 ] \
    && mkdir -p $ZK_DATA_DIR $ZK_DATA_LOG_DIR $ZK_LOG_DIR /usr/share/zookeeper /tmp/zookeeper /usr/etc/ \
    && chown -R "$ZK_USER:$ZK_USER" /opt/$ZK_DIST $ZK_DATA_DIR $ZK_LOG_DIR $ZK_DATA_LOG_DIR /tmp/zookeeper \
    && ln -s /opt/zookeeper/conf/ /usr/etc/zookeeper \
    && ln -s /opt/zookeeper/bin/* /usr/bin \
    && ln -s /opt/zookeeper/$ZK_DIST.jar /usr/share/zookeeper/ \
    && ln -s /opt/zookeeper/lib/* /usr/share/zookeeper

CMD bootstrap.sh
