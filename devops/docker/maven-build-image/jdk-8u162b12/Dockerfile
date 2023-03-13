FROM adoptopenjdk:17-jdk-hotspot

############################# Install tools and deps #########################################
 RUN apt-get update && apt-get install -y curl tar bash procps git openssh-client htop xmlstarlet libxslt1-dev && rm -rf /var/lib/apt/lists/*
############################# End Install tools and deps #####################################

############################# Install Groovy ################################
ENV GROOVY_HOME /opt/groovy
ARG GROOVY_VERSION=3.0.9

RUN set -o errexit -o nounset && echo "Installing build dependencies" && \
    apt-get update && apt-get install -y --no-install-recommends gnupg2 ca-certificates && \
    echo "Downloading Groovy" && wget -O groovy.zip "https://dl.bintray.com/groovy/maven/apache-groovy-binary-${GROOVY_VERSION}.zip" && \
    echo "Importing keys listed in https://dist.apache.org/repos/dist/release/groovy/KEYS from key server" && export GNUPGHOME="$(mktemp -d)" && for key in \
		"602DBB08EE936C7F" \
        "5D2C85F28F9B4859" \
	; do \
		for server in \
			"keyserver.ubuntu.com" \
            "hkp://keyserver.ubuntu.com:80" \
		; do \
			echo "  Trying ${server}"; \
			if gpg --keyserver "${server}" --recv-keys "${key}"; then \
				break; \
			fi; \
		done; \
	done; \
	if [ $(gpg --list-keys | grep -c "pub ") -ne 5 ]; then \
		echo "ERROR: Failed to fetch GPG keys" >&2; \
		exit 1; \
	fi \
	&& echo "Checking download signature" \
	&& wget -O groovy.zip.asc "https://dl.bintray.com/groovy/maven/apache-groovy-binary-${GROOVY_VERSION}.zip.asc" \
	&& gpg --batch --verify groovy.zip.asc groovy.zip && rm -rf "${GNUPGHOME}" && rm groovy.zip.asc \
	&& echo "Installing Groovy" \
	&& unzip groovy.zip \
	&& rm groovy.zip \
	&& mkdir -p /opt \
	&& mv "groovy-${GROOVY_VERSION}" "${GROOVY_HOME}/" \
	&& ln -s "${GROOVY_HOME}/bin/grape" /usr/bin/grape \
	&& ln -s "${GROOVY_HOME}/bin/groovy" /usr/bin/groovy \
	&& ln -s "${GROOVY_HOME}/bin/groovyc" /usr/bin/groovyc \
	&& ln -s "${GROOVY_HOME}/bin/groovyConsole" /usr/bin/groovyConsole \
	&& ln -s "${GROOVY_HOME}/bin/groovydoc" /usr/bin/groovydoc \
	&& ln -s "${GROOVY_HOME}/bin/groovysh" /usr/bin/groovysh \
	&& ln -s "${GROOVY_HOME}/bin/java2groovy" /usr/bin/java2groovy \
	&& echo "Cleaning up build dependencies" && apt-get purge -y --auto-remove gnupg2 \
	&& echo "Symlinking root .groovy to groovy .groovy" \
	&& ln -s /home/groovy/.groovy /root/.groovy
############################# End Install Groovy ################################

############################# Install Maven ################################

ARG MAVEN_VERSION=3.5.3
ARG USER_HOME_DIR="/root"
ARG SHA=b52956373fab1dd4277926507ab189fb797b3bc51a2a267a193c931fffad8408
ARG BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries

RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  && echo "${SHA}  /tmp/apache-maven.tar.gz" | sha256sum -c - \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"
ENV M2_REPO "$MAVEN_CONF/repository"
############################# End Install Maven ################################

ENV CHECKSTYLE_HOME /usr/local/checkstyle/
WORKDIR $CHECKSTYLE_HOME
RUN set -x && cd $CHECKSTYLE_HOME/.. && \
    git clone https://github.com/checkstyle/checkstyle.git && cd checkstyle && \
    mvn dependency:go-offline && \
    mvn clean site -Pno-validations -Dmaven.test.failure.ignore=true && \
    mvn verify -DskipTests -Dmaven.test.failure.ignore=true || true && \
    mvn org.pitest:pitest-maven:mutationCoverage -DskipTests -Dmaven.test.failure.ignore=true || true && \
    # We don't fail the build on PMD/pitest/etc. warnings because the goal is to prefetch PMD/pitest/etc. Maven plugins into the image \
    rm -rf $CHECKSTYLE_HOME/*

CMD [ "echo", "To start build, run 'mvn ...' to start some Checkstyle build here" ]
