FROM openjdk:8-jdk
LABEL MAINTAINER="Jack T. Jia <jack-tiefeng.jia@ibm.com>"

#####################################################
# arguments
ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000
ARG JENKINS_AGENT_HOME=/home/${user}
ARG JNLP_SLAVE_VERSION=3.28
ARG NODE_VERSION=8.11.4
ARG FIREFOX_VERSION=61.0.2
ARG FIREFOX_LANGUAGE=en-US

#####################################################
# environments
ENV DEBIAN_FRONTEND noninteractive
ENV JENKINS_AGENT_HOME ${JENKINS_AGENT_HOME}
ENV JNLP_SLAVE_VERSION ${JNLP_SLAVE_VERSION}
ENV NODE_VERSION ${NODE_VERSION}
ENV NVM_VERSION 0.33.11
ENV CHECKLINK_VERSION 4_81
ENV DOCKER_VERSION 18.09.0
ENV DOCKER_CHANNEL stable
ENV DOCKER_ARCH x86_64

#####################################################
# create jenkins user
RUN groupadd -g ${gid} ${group} \
    && useradd -d "${JENKINS_AGENT_HOME}" -u "${uid}" -g "${gid}" -m -s /bin/bash "${user}"
RUN mkdir ${JENKINS_AGENT_HOME}/.jenkins

#####################################################
# install required packages
# - build-essential: provide make and gcc which will be used in "npm install"
# - sshpass: allow ssh to other servers
# - bzip2: used by installing firefox
# - gnome-keyring: required by keytar
# - libsecret-1-dev: required by npm install rebuild keytar
# - dbus-x11: includes dbus-launch
# - libdbus-glib-1-2: used by firefox
# - cpanminus libssl-dev: used by w3c link checker
# - sudo: required by docker-in-docker
RUN apt-get update && apt-get install --no-install-recommends -y \
    openssh-server \
    vim curl wget rsync pax build-essential sshpass bzip2 jq locales \
    cpanminus libssl-dev \
    gnome-keyring libsecret-1-dev dbus dbus-user-session dbus-x11 \
    libdbus-glib-1-2 \
    sudo \
   && rm -rf /var/lib/apt/lists/*

#####################################################
# configure locale
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
    && /usr/sbin/locale-gen

#####################################################
# install jnlp slave jar
RUN curl --create-dirs -fsSLo /usr/share/jenkins/slave.jar https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/${JNLP_SLAVE_VERSION}/remoting-${JNLP_SLAVE_VERSION}.jar \
  && chmod 755 /usr/share/jenkins \
  && chmod 644 /usr/share/jenkins/slave.jar

#####################################################
# install firefox
ADD https://archive.mozilla.org/pub/firefox/releases/${FIREFOX_VERSION}/KEY KEY
ADD https://archive.mozilla.org/pub/firefox/releases/${FIREFOX_VERSION}/SHA512SUMS SHA512SUMS
ADD https://archive.mozilla.org/pub/firefox/releases/${FIREFOX_VERSION}/SHA512SUMS.asc SHA512SUMS.asc
RUN gpg --no-tty --import KEY \
    && gpg --no-tty --verify SHA512SUMS.asc \
    && rm KEY \
    && rm SHA512SUMS.asc
# need RUN rather than ADD or COPY because both ADD and COPY are silently unzipping the archive
RUN wget --no-verbose --show-progress --progress=dot:giga --directory-prefix linux-x86_64/${FIREFOX_LANGUAGE} https://archive.mozilla.org/pub/firefox/releases/${FIREFOX_VERSION}/linux-x86_64/${FIREFOX_LANGUAGE}/firefox-${FIREFOX_VERSION}.tar.bz2 \
    && grep linux-x86_64/${FIREFOX_LANGUAGE}/firefox-${FIREFOX_VERSION}.tar.bz2 SHA512SUMS | sha512sum -c - \
    && rm SHA512SUMS \
    && tar --extract --bzip2 --file linux-x86_64/${FIREFOX_LANGUAGE}/firefox-${FIREFOX_VERSION}.tar.bz2 --directory /usr/lib/ \
    && rm -fr linux-x86_64 \
    && ln -fs /usr/lib/firefox/firefox /usr/bin/firefox

#####################################################
# setup SSH server
RUN sed -i /etc/ssh/sshd_config \
        -e 's/#PermitRootLogin.*/PermitRootLogin no/' \
        -e 's/#RSAAuthentication.*/RSAAuthentication yes/'  \
        -e 's/#PasswordAuthentication.*/PasswordAuthentication no/' \
        -e 's/#SyslogFacility.*/SyslogFacility AUTH/' \
        -e 's/#LogLevel.*/LogLevel INFO/' && \
    mkdir /var/run/sshd

#####################################################
# Copy the PAM configuration options to allow auto unlocking of the gnome keyring
COPY pam.config /tmp/pam.config
# Enable unlocking for ssh/login
RUN cat /tmp/pam.config >> /etc/pam.d/sshd \
    && cat /tmp/pam.config >> /etc/pam.d/login \
    && rm /tmp/pam.config

COPY .bashrc_all /tmp/.bashrc_all
COPY .bashrc_ni /home/jenkins/.bashrc_ni
# prepend to ~/.bashrc
RUN sed -i -e "/# If not running interactively, don't do anything/r /tmp/.bashrc_all" -e //N /home/jenkins/.bashrc

#####################################################
# install w3c link checker
RUN set -x \
  && curl -sSL https://github.com/w3c/link-checker/archive/checklink-${CHECKLINK_VERSION}.tar.gz -o /tmp/link-checker.tar.gz \
  && tar -xzf /tmp/link-checker.tar.gz -C /tmp \
  && rm /tmp/link-checker.tar.gz \
  && cd /tmp/link-checker-checklink-${CHECKLINK_VERSION} \
  && cpanm --installdeps . \
  && cpanm LWP::Protocol::https \
  && perl Makefile.PL \
  && make \
  && make test \
  && make install \
  && rm -rf /tmp/link-checker-checklink-${CHECKLINK_VERSION}

#####################################################
# install docker
RUN set -eux; \
  groupadd docker; \
  useradd -g docker docker; \
  usermod -aG docker ${user}; \
  if ! wget -O docker.tgz "https://download.docker.com/linux/static/${DOCKER_CHANNEL}/${DOCKER_ARCH}/docker-${DOCKER_VERSION}.tgz"; then \
    echo >&2 "error: failed to download 'docker-${DOCKER_VERSION}' from '${DOCKER_CHANNEL}' for '${DOCKER_ARCH}'"; \
    exit 1; \
  fi; \
  \
  tar --extract \
    --file docker.tgz \
    --strip-components 1 \
    --directory /usr/local/bin/ \
  ; \
  rm docker.tgz; \
  \
  dockerd --version; \
  docker --version

#####################################################
# allow jenkins user to use sudo without password
RUN (echo "${user} ALL = (ALL) NOPASSWD: ALL" > /etc/sudoers.d/jenkinsnosudo && \
  chmod 0440 /etc/sudoers.d/jenkinsnosudo)

#####################################################
# install nvm on jenkins user
# switch to jenkins user
USER ${user}
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v${NVM_VERSION}/install.sh | bash
ENV NVM_DIR /home/${user}/.nvm
# install node version and set it as the default one
RUN /bin/bash -c "source ${NVM_DIR}/nvm.sh && nvm install $NODE_VERSION && nvm alias default $NODE_VERSION && nvm use default && npm install -g yarn && npm install -g jfrog-cli-go"
ENV NODE_PATH ${NVM_DIR}/versions/node/v${NODE_VERSION}/lib/node_modules
ENV PATH      ${NVM_DIR}/versions/node/v${NODE_VERSION}/bin:$PATH

#####################################################
# define volume
VOLUME "${JENKINS_AGENT_HOME}" "/tmp" "/run" "/var/run"
WORKDIR "${JENKINS_AGENT_HOME}"
# switch back to root user
USER root

#####################################################
# expose and entrypoint
EXPOSE 22
COPY setup-entrypoint /usr/local/bin/setup-entrypoint
RUN chmod +x /usr/local/bin/setup-entrypoint
ENTRYPOINT ["/usr/local/bin/setup-entrypoint"]
