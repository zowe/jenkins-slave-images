FROM node:6.14.4-alpine
LABEL MAINTAINER="Jack T. Jia <jack-tiefeng.jia@ibm.com>"


ENV APP_DIR /app

# set user
USER root
RUN mkdir ${APP_DIR}
WORKDIR ${APP_DIR}

#####################################################
# install required packages
# /dev/urandom is used as random source, which is perfectly safe
# according to http://www.2uo.de/myths-about-urandom/
RUN apk add --update \
    bash \
    curl \
    openjdk8-jre-base=8.111.14-r0 \
    nginx \
  && rm /var/cache/apk/* \
  && echo "securerandom.source=file:/dev/urandom" >> /usr/lib/jvm/default-jvm/jre/lib/security/java.security

#####################################################
# preset nginx config
COPY nginx.conf /etc/nginx/nginx.conf

#####################################################
# expose and entrypoint
EXPOSE 80
COPY setup-entrypoint /usr/local/bin/setup-entrypoint
RUN chmod +x /usr/local/bin/setup-entrypoint
ENTRYPOINT ["/usr/local/bin/setup-entrypoint"]
