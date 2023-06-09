# metabase-arm64
Metabase for ARM64

Taken from https://github.com/metabase/metabase/issues/13119 

## Getting Started

- Clone this repository
- Make sure you're in the directory with the `Dockerfile`.
- Build the image:

```
docker build -t yourimagename .
```

- Run the container:
```
docker run -d -p 3000:3000 --name metabase yourimagename
```

## `segfault` or other errors

Consider modifying the Dockerfile to look something more like this:

```
FROM ubuntu:21.04

ENV FC_LANG en-US LC_CTYPE en_US.UTF-8

# dependencies
RUN apt-get update -yq && apt-get install -yq bash fonts-dejavu-core fonts-dejavu-extra fontconfig curl openjdk-11-jre-headless && \
    apt-get clean && \
    rm -rf /var/lib/{apt,dpkg,cache,log}/ && \
    mkdir -p /app/certs && \
    curl https://s3.amazonaws.com/rds-downloads/rds-combined-ca-bundle.pem -o /app/certs/rds-combined-ca-bundle.pem  && \
    keytool -noprompt -import -trustcacerts -alias aws-rds -file /app/certs/rds-combined-ca-bundle.pem -keystore /etc/ssl/certs/java/cacerts -keypass changeit -storepass changeit && \
    curl https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem -o /app/certs/DigiCertGlobalRootG2.crt.pem  && \
    keytool -noprompt -import -trustcacerts -alias azure-cert -file /app/certs/DigiCertGlobalRootG2.crt.pem -keystore /etc/ssl/certs/java/cacerts -keypass changeit -storepass changeit && \
    mkdir -p /plugins && chmod a+rwx /plugins && \
    useradd --shell /bin/bash metabase


WORKDIR /app

# copy app from the offical image
COPY --from=metabase/metabase:latest /app /app

RUN chown -R metabase /app

USER metabase
# expose our default runtime port
EXPOSE 3000

# run it
ENTRYPOINT ["/app/run_metabase.sh"]
```
- Access Metabase at `http://<IP of your machine>:3000` and run through the configuration prompts

- Making sure to specify Ubuntu version, etc. 
