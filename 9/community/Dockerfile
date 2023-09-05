FROM eclipse-temurin:17-jre
# FROM --platform=linux/amd64 eclipse-temurin:17-jre

LABEL org.opencontainers.image.url=https://github.com/SonarSource/docker-sonarqube

ENV LANG='en_US.UTF-8' \
    LANGUAGE='en_US:en' \
    LC_ALL='en_US.UTF-8'

#
# SonarQube setup
#
ARG SONARQUBE_VERSION=9.9.1.69595
ARG SONARQUBE_ZIP_URL=https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-${SONARQUBE_VERSION}.zip
ENV JAVA_HOME='/opt/java/openjdk' \
    SONARQUBE_HOME=/opt/sonarqube \
    SONAR_VERSION="${SONARQUBE_VERSION}" \
    SQ_DATA_DIR="/opt/sonarqube/data" \
    SQ_EXTENSIONS_DIR="/opt/sonarqube/extensions" \
    SQ_LOGS_DIR="/opt/sonarqube/logs" \
    SQ_TEMP_DIR="/opt/sonarqube/temp"

RUN set -eux; \
    groupadd --system --gid 1000 sonarqube; \
    useradd -m --system --uid 1000 --gid sonarqube sonarqube; \
    apt-get update; \
    apt-get install -y gnupg unzip curl bash fonts-dejavu xz-utils apt-transport-https git; \
    echo "networkaddress.cache.ttl=5" >> "${JAVA_HOME}/conf/security/java.security"; \
    sed --in-place --expression="s?securerandom.source=file:/dev/random?securerandom.source=file:/dev/urandom?g" "${JAVA_HOME}/conf/security/java.security"; \
    # pub   2048R/D26468DE 2015-05-25
    #       Key fingerprint = F118 2E81 C792 9289 21DB  CAB4 CFCA 4A29 D264 68DE
    # uid                  sonarsource_deployer (Sonarsource Deployer) <infra@sonarsource.com>
    # sub   2048R/06855C1D 2015-05-25
    for server in $(shuf -e hkps://keys.openpgp.org \
                            hkps://keyserver.ubuntu.com) ; do \
        gpg --batch --keyserver "${server}" --recv-keys 679F1EE92B19609DE816FDE81DB198F93525EC1A && break || : ; \
    done; \
    mkdir --parents /opt; \
    cd /opt; \
    curl --fail --location --output sonarqube.zip --silent --show-error "${SONARQUBE_ZIP_URL}"; \
    curl --fail --location --output sonarqube.zip.asc --silent --show-error "${SONARQUBE_ZIP_URL}.asc"; \
    gpg --batch --verify sonarqube.zip.asc sonarqube.zip; \
    unzip -q sonarqube.zip; \
    mv "sonarqube-${SONARQUBE_VERSION}" sonarqube; \
    rm sonarqube.zip*; \
    rm -rf ${SONARQUBE_HOME}/bin/*; \
    ln -s "${SONARQUBE_HOME}/lib/sonar-application-${SONARQUBE_VERSION}.jar" "${SONARQUBE_HOME}/lib/sonarqube.jar"; \
    chmod -R 555 ${SONARQUBE_HOME}; \
    chmod -R ugo+wrX "${SQ_DATA_DIR}" "${SQ_EXTENSIONS_DIR}" "${SQ_LOGS_DIR}" "${SQ_TEMP_DIR}"; \
    curl --fail --location --output sonar-flutter-plugin.jar --silent --show-error https://github.com/insideapp-oss/sonar-flutter/releases/download/0.5.0/sonar-flutter-plugin-0.5.0.jar; \
    mv sonar-flutter-plugin.jar  "${SONARQUBE_HOME}/extensions/plugins/"; \
    curl --fail --location --output flutter_linux_stable.tar.xz --silent --show-error https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_3.13.2-stable.tar.xz; \
    tar xf flutter_linux_stable.tar.xz; \
    rm flutter_linux_stable.tar.xz; \
    export PATH="$PATH:/opt/flutter/flutter/bin"; \
    # wget -qO- https://dl-ssl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/dart.gpg; \
    # echo 'deb [signed-by=/usr/share/keyrings/dart.gpg arch=amd64] https://storage.googleapis.com/download.dartlang.org/linux/debian stable main' | tee /etc/apt/sources.list.d/dart_stable.list; \
    # apt-get update; \
    # apt-get install dart; \
    # curl --fail --location --output dart_amd64.deb --silent --show-error https://storage.googleapis.com/dart-archive/channels/stable/release/latest/linux_packages/dart_3.1.0-1_amd64.deb; \
    # dpkg -i dart_amd64.deb; \
    # rm dart_amd64.deb; \
    # export PATH="$PATH:/usr/lib/dart/bin"; \
    echo export PATH="$PATH:/opt/flutter/bin" >> /home/sonarqube/.bashrc; \
    chown sonarqube:sonarqube -R /opt/flutter; \
    rm -rf /var/lib/apt/lists/*; \
    apt-get remove -y gnupg unzip curl;

ENV flutter="/opt/flutter/bin/flutter"

COPY entrypoint.sh ${SONARQUBE_HOME}/docker/

WORKDIR ${SONARQUBE_HOME}
EXPOSE 9000

USER sonarqube
STOPSIGNAL SIGINT

ENTRYPOINT ["/opt/sonarqube/docker/entrypoint.sh"]
