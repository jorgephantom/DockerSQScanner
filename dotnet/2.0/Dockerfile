FROM openjdk:8-jre-slim

ENV SONAR_SCANNER_MSBUILD_VERSION=4.1.0.1148 \
    SONAR_SCANNER_VERSION=3.1.0.1141 \
    SONAR_SCANNER_MSBUILD_HOME=/opt/sonar-scanner-msbuild \
    WHITESOURCE_HOME=/opt/whitesource \
    DOTNET_PROJECT_DIR=/project \
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE=true \
    DOTNET_CLI_TELEMETRY_OPTOUT=true \
    DOTNET_SDK_VERSION=2.1.202 \
    DOTNET_SDK_DOWNLOAD_URL=https://dotnetcli.blob.core.windows.net/dotnet/Sdk/2.1.202/dotnet-sdk-2.1.202-linux-x64.tar.gz \
	DOTNET_SDK_DOWNLOAD_SHA=e785b9b488b5570708eb060f9a4cb5cf94597d99a8b0a3ee449d2e5df83771c1ba643a87db17ae6727d0e2acb401eca292fb8c68ad92eeb59d7f0d75eab1c20a \
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true \
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true \
    # Skip extraction of XML docs - generally not useful within an image/container - helps perfomance
    NUGET_XMLDOC_MODE=skip \
    RestoreUseSkipNonexistentTargets=false \
    CF_BGD_VERSION="1.3.0" \
    CF_BGD_CHECKSUM="c74995ae0ba3ec9eded9c2a555e5984ba536d314cf9dc30013c872eb6b9d76b6" \
    CF_BGD_TEMPFILE="/tmp/blue-green-deploy.linux64"

RUN set -x \
  && apt-get update \
  && apt-get install -y --no-install-recommends \
    gnupg \
    dirmngr \
    curl \
    gettext \
    wget \
    libc6 \
    libcurl3 \
    libgcc1 \
    libgssapi-krb5-2 \
    libicu57 \
    liblttng-ust0 \
    libssl1.0.2 \
    libstdc++6 \
    libunwind8 \
    libuuid1 \
    zlib1g \
  && rm -rf /var/lib/apt/lists/* \
  && curl -SL "${DOTNET_SDK_DOWNLOAD_URL}" --output dotnet.tar.gz \
  && echo "${DOTNET_SDK_DOWNLOAD_SHA} dotnet.tar.gz" | sha512sum -c - \
  && mkdir -p /usr/share/dotnet \
  && tar -zxf dotnet.tar.gz -C /usr/share/dotnet \
  && rm dotnet.tar.gz \
  && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet \
  && mkdir warmup \
  && cd warmup \
  && dotnet new \
  && cd .. \
  && rm -rf warmup \
  && rm -rf /tmp/NuGetScratch

RUN wget https://github.com/SonarSource/sonar-scanner-msbuild/releases/download/$SONAR_SCANNER_MSBUILD_VERSION/sonar-scanner-msbuild-$SONAR_SCANNER_MSBUILD_VERSION-netcoreapp2.0.zip -O /opt/sonar-scanner-msbuild.zip \
  && mkdir -p $SONAR_SCANNER_MSBUILD_HOME \
  && mkdir -p $DOTNET_PROJECT_DIR \
  && unzip /opt/sonar-scanner-msbuild.zip -d $SONAR_SCANNER_MSBUILD_HOME \
  && rm /opt/sonar-scanner-msbuild.zip \
  && chmod 775 $SONAR_SCANNER_MSBUILD_HOME/*.dll \
  && chmod 775 $SONAR_SCANNER_MSBUILD_HOME/**/bin/* \
  && chmod 775 $SONAR_SCANNER_MSBUILD_HOME/**/lib/*.jar
  
RUN mkdir -p $WHITESOURCE_HOME
ADD https://s3.amazonaws.com/file-system-agent/whitesource-fs-agent-18.2.1.jar $WHITESOURCE_HOME
RUN chmod +x $WHITESOURCE_HOME/whitesource-fs-agent-18.2.1.jar
  
# Install Cloud Foundry cli
ADD https://cli.run.pivotal.io/stable?release=linux64-binary&version=6.32.0 /tmp/cf-cli.tgz
RUN mkdir -p /usr/local/bin && \
  tar -xzf /tmp/cf-cli.tgz -C /usr/local/bin && \
  cf --version && \
  rm -f /tmp/cf-cli.tgz
  
RUN curl -L -o "${CF_BGD_TEMPFILE}" \
  "https://github.com/bluemixgaragelondon/cf-blue-green-deploy/releases/download/v${CF_BGD_VERSION}/blue-green-deploy.linux64" \
  && echo "${CF_BGD_CHECKSUM}  ${CF_BGD_TEMPFILE}" | sha256sum -c - \
  && chmod +x "${CF_BGD_TEMPFILE}" \
  && cf install-plugin -f "${CF_BGD_TEMPFILE}" \
  && rm "${CF_BGD_TEMPFILE}"

# Install cf cli Autopilot plugin
ADD https://github.com/contraband/autopilot/releases/download/0.0.4/autopilot-linux /tmp/autopilot-linux
RUN chmod +x /tmp/autopilot-linux && \
  cf install-plugin /tmp/autopilot-linux -f && \
  rm -f /tmp/autopilot-linux

ENV PATH="$SONAR_SCANNER_MSBUILD_HOME:$SONAR_SCANNER_MSBUILD_HOME/sonar-scanner-$SONAR_SCANNER_VERSION/bin:${PATH}"