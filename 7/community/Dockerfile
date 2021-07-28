FROM openjdk:11-jre-slim

RUN apt-get update \
    && apt-get install -y curl gnupg2 unzip \
    && rm -rf /var/lib/apt/lists/*

ENV SONAR_VERSION=7.9.6 \
    SONARQUBE_HOME=/opt/sonarqube \
    SONARQUBE_JDBC_USERNAME=sonar \
    SONARQUBE_JDBC_PASSWORD=sonar \
    SONARQUBE_JDBC_URL=""

# Http port
EXPOSE 9000

RUN groupadd -r sonarqube && useradd -r -g sonarqube sonarqube

# pub   2048R/D26468DE 2015-05-25
#       Key fingerprint = F118 2E81 C792 9289 21DB  CAB4 CFCA 4A29 D264 68DE
# uid                  sonarsource_deployer (Sonarsource Deployer) <infra@sonarsource.com>
# sub   2048R/06855C1D 2015-05-25
RUN for server in $(shuf -e ha.pool.sks-keyservers.net \
                            hkp://p80.pool.sks-keyservers.net:80 \
                            keyserver.ubuntu.com \
                            hkp://keyserver.ubuntu.com:80 \
                            pgp.mit.edu) ; do \
        gpg --batch --keyserver "$server" --recv-keys F1182E81C792928921DBCAB4CFCA4A29D26468DE && break || : ; \
    done

RUN set -x \
    && cd /opt \
    && curl -o sonarqube.zip -fSL https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-$SONAR_VERSION.zip \
    && curl -o sonarqube.zip.asc -fSL https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-$SONAR_VERSION.zip.asc \
    && gpg --batch --verify sonarqube.zip.asc sonarqube.zip \
    && unzip -q sonarqube.zip \
    && mv sonarqube-$SONAR_VERSION sonarqube \
    && chown -R sonarqube:sonarqube sonarqube \
    && rm sonarqube.zip* \
    && rm -rf $SONARQUBE_HOME/bin/*

# Community Branch plugin
ADD --chown=10001:10001 https://github.com/mc1arke/sonarqube-community-branch-plugin/releases/download/1.3.0/sonarqube-community-branch-plugin-1.3.0.jar /opt/sonarqube/extensions/plugins/
ADD --chown=10001:10001 https://github.com/mc1arke/sonarqube-community-branch-plugin/releases/download/1.3.0/sonarqube-community-branch-plugin-1.3.0.jar /opt/sonarqube/lib/common/

# Mutation Analysis results plugin
ADD --chown=10001:10001 https://github.com/devcon5io/mutation-analysis-plugin/releases/download/v1.5/mutation-analysis-plugin-1.5.jar /opt/sonarqube/extensions/plugins/

# ShellCheck plugin
ADD --chown=10001:10001 https://github.com/sbaudoin/sonar-shellcheck/releases/download/v2.4.0/sonar-shellcheck-plugin-2.4.0.jar /opt/sonarqube/extensions/plugins/


VOLUME "$SONARQUBE_HOME/data"

WORKDIR $SONARQUBE_HOME
COPY run.sh $SONARQUBE_HOME/bin/
USER sonarqube
ENTRYPOINT ["./bin/run.sh"]
