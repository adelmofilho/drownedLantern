FROM adelmofilho/r-base:3.6.2

LABEL org.label-schema.license="GPL-3.0" \
      org.label-schema.vcs-url="https://github.com/adelmofilho/Whitesnake/rstudio/1.2.5033" \
      org.label-schema.vendor="Hope Clouds Observation" \
      maintainer="Adelmo Filho <adelmo.aguiar.filho@gmail.com>"

# Assumir defaults nas instalações 
ENV DEBIAN_FRONTEND=noninteractive

# Criação do usuário para rstudio-server
ENV USER rstudio
ENV PASSWORD rstudio

RUN useradd rstudio \
  && echo "${USER}:${PASSWORD}" | chpasswd \
  && mkdir /home/${USER} \
  && chown ${USER}:${USER} /home/${USER}

# Instalar dependências
RUN apt-get update && apt-get install -y --no-install-recommends \
    gdebi-core \ 
    lsb-release \
    libapparmor1 \ 
    libedit2 \
    libclang-dev \
    sudo \
    psmisc \
    qpdf \
&& rm -rf /var/lib/apt/lists/*

# Instala rstudio-server
RUN wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-1.2.5033-amd64.deb

RUN gdebi -n rstudio-server-1.2.5033-amd64.deb

RUN mkdir -p /etc/R \
  && echo 'if(is.na(Sys.getenv("HTTR_LOCALHOST", unset=NA))) { options(httr_oob_default = TRUE) }' >> /usr/local/lib/R/etc/Rprofile.site \
  && echo "PATH=${PATH}" >> /usr/local/lib/R/etc/Renviron \
  && echo 'rsession-which-r=/usr/local/bin/R' >> /etc/rstudio/rserver.conf \
  && echo 'lock-type=advisory' >> /etc/rstudio/file-locks

# Configura sistema s6 para inicialização do rstudio-server com o container
RUN git config --system credential.helper 'cache --timeout=3600' \
 && git config --system push.default simple \
 && wget -P /tmp/ https://github.com/just-containers/s6-overlay/releases/download/v1.21.7.0/s6-overlay-amd64.tar.gz \
 && tar xzf /tmp/s6-overlay-amd64.tar.gz -C / \
 && mkdir -p /etc/services.d/rstudio \
 && echo "#!/usr/bin/with-contenv bash \
          \n## load /etc/environment vars first: \
  		  \n for line in $( cat /etc/environment ) ; do export $line ; done \
          \n exec /usr/lib/rstudio-server/bin/rserver --server-daemonize 0" \
          > /etc/services.d/rstudio/run \
 && echo "#!/bin/bash \
          \n rstudio-server stop" \
          > /etc/services.d/rstudio/finish \
 && mkdir -p /home/rstudio/.rstudio/monitored/user-settings \
 && echo "alwaysSaveHistory='0' \
          \nloadRData='0' \
          \nsaveAction='0'" \
          > /home/rstudio/.rstudio/monitored/user-settings/user-settings \
 && chown -R rstudio:rstudio /home/rstudio/.rstudio

RUN chmod 777 /home

CMD ["/init"]
