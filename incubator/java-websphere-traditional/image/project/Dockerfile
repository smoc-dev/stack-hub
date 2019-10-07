FROM registry.access.redhat.com/ubi7/ubi as compile

RUN yum upgrade --disableplugin=subscription-manager -y \
   && yum clean --disableplugin=subscription-manager packages \
   && mkdir -p /mvn/repository \
   && echo 'Finished installing dependencies'

# Dependency install
RUN yum install --disableplugin=subscription-manager -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm \
   && yum install --disableplugin=subscription-manager -y curl unzip xmlstarlet wget ca-certificates

#Install openjdk
ENV JAVA_VERSION jdk8u222-b10_openj9-0.15.1

RUN set -eux; \
   ESUM='20cff719c6de43f8bb58c7f59e251da7c1fa2207897c9a4768c8c669716dc819'; \
   BINARY_URL='https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u222-b10_openj9-0.15.1/OpenJDK8U-jdk_x64_linux_openj9_8u222b10_openj9-0.15.1.tar.gz'; \
   curl -LfsSo /tmp/openjdk.tar.gz ${BINARY_URL}; \
   echo "${ESUM} */tmp/openjdk.tar.gz" | sha256sum -c -; \
   mkdir -p /opt/java/openjdk; \
   cd /opt/java/openjdk; \
   tar -xf /tmp/openjdk.tar.gz --strip-components=1; \
   rm -rf /tmp/openjdk.tar.gz;

   ENV JAVA_HOME=/opt/java/openjdk \
   PATH="/opt/java/openjdk/bin:$PATH"
   ENV JAVA_TOOL_OPTIONS="-XX:+IgnoreUnrecognizedVMOptions -XX:+UseContainerSupport -XX:+IdleTuningCompactOnIdle -XX:+IdleTuningGcOnIdle"

# Maven install
ARG MAVEN_VERSION=3.6.2
ARG USER_HOME_DIR="/root"
ARG SHA=d941423d115cd021514bfd06c453658b1b3e39e6240969caf4315ab7119a77299713f14b620fb2571a264f8dff2473d8af3cb47b05acf0036fc2553199a5c1ee
ARG BASE_URL=https://archive.apache.org/dist/maven/maven-3/${MAVEN_VERSION}/binaries/

RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
 && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
 && echo "${SHA}  /tmp/apache-maven.tar.gz" | sha512sum -c - \
 && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
 && rm -f /tmp/apache-maven.tar.gz \
 && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

#setup project folder for java build step
COPY . /project
COPY ./mvn-stack-settings.xml /usr/share/maven/conf/settings.xml

# OSGI core for version range processing
RUN  /project/util/check_version build

WORKDIR /project/user-app

RUN /project/java-twas-build.sh package

####
FROM ibmcom/websphere-traditional:9.0.5.1-ubi

WORKDIR /

COPY --from=compile /project/user-app/app/target/app.ear /work/config/user-app/
# COPY --from=compile /project/user-app/app.ear /work/config/user-app/
COPY --from=compile /project/install_app_prod.py /work/config/install_app.py
COPY --from=compile /project/was-config.props /work/config/was-config.props

RUN /work/configure.sh
