# Step 1: Build the user's application
FROM {{.stack.baseimage}} as compile

RUN mkdir -p /mvn/repository

#setup project folder for java build step
COPY . /project
COPY ./mvn-stack-settings.xml /usr/share/maven/conf/settings.xml

WORKDIR /project/user-app

RUN /project/util/check_version build \
 && /project/java-twas-build.sh package

####

# Step 2: Package tWAS image
FROM {{.stack.finalimage}}

WORKDIR /

COPY --from=compile /project/user-app/config/ /work/config/

COPY --from=compile /project/user-app/app/target/app.ear /work/config/user-app/
# COPY --from=compile /project/user-app/app.ear /work/config/user-app/

RUN /work/configure.sh
