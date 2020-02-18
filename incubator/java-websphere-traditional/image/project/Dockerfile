FROM kabanero/ubi8-maven as compile

RUN mkdir -p /mvn/repository

#setup project folder for java build step
COPY . /project
COPY ./mvn-stack-settings.xml /usr/share/maven/conf/settings.xml

# OSGI core for version range processing
RUN  /project/util/check_version build

WORKDIR /project/user-app

RUN /project/java-twas-build.sh package

####
FROM ibmcom/websphere-traditional:9.0.5.2-ubi

WORKDIR /

COPY --from=compile /project/user-app/app/target/app.ear /work/config/user-app/
# COPY --from=compile /project/user-app/app.ear /work/config/user-app/
COPY --from=compile /project/install_app_prod.py /work/config/install_app.py
COPY --from=compile /project/was-config.props /work/config/was-config.props

RUN /work/configure.sh
