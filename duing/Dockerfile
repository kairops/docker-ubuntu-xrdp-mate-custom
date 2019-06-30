FROM rattydave/docker-ubuntu-xrdp-mate-custom:19.04

RUN apt-get update -y && \ 
    apt-get install -yqq privoxy chromium-browser chromium-chromedriver gcc make ruby ruby-dev rubygems zlibc zlib1g zlib1g-dev libxml2 libxml2-dev gedit git npm firefox && \
    wget -qO - https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg | apt-key add - && \
    echo 'deb https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/repos/debs/ vscodium main' >> /etc/apt/sources.list && \
    apt-get update -y && \
    apt-get install -y codium && \
    apt-get -y autoclean && apt-get -y autoremove && \
    apt-get -y purge $(dpkg --get-selections | grep deinstall | sed s/deinstall//g) && \
    rm -rf /var/lib/apt/lists/*
    
RUN gem install rake && \
    gem install bddfire

ADD gherkin.lang /usr/share/gtksourceview-3.0/language-specs/gherkin.lang

RUN useradd -ms /bin/bash ubuntu && \
    echo "ubuntu:ubuntu" | chpasswd && \
    usermod -aG sudo ubuntu

RUN su - ubuntu -c "npm config set prefix '~/.npm-global'" && \
    su - ubuntu -c "echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc" && \
    su - ubuntu -c "npm install -g phantomjs-prebuilt geckodriver"

RUN git clone https://github.com/red-panda-ci/ci-scripts /opt/duingdemo/ci-scripts/common && \
    cd /opt/duingdemo && \
    bash ci-scripts/common/bin/_install.sh cucumber && \
    cd ci-scripts/test/cucumber && \
    bundle install && \
    sed "s/ddefault_max_wait_time/default_max_wait_time/g" -i features/support/env.rb && \
    sed "s#--load-images=no#--load-images=yes#g" -i features/support/env.rb && \
    sed "s#www.google.com#www.google.com?hl=en#g" -i features/bddfire.feature && \
    cp config.yml.dist config.yml && \
    chown -R ubuntu:ubuntu /opt/duingdemo

RUN sed -i "s#listen-address  \[::1\]:8118##g" /etc/privoxy/config && \
    echo "debug 1" >> /etc/privoxy/config && \
    echo "[program:privoxy]" > /etc/supervisor/conf.d/privoxy.conf && \
    echo "command=/usr/sbin/privoxy --no-daemon /etc/privoxy/config" >> /etc/supervisor/conf.d/privoxy.conf && \
    echo "process_name = privoxy" >> /etc/supervisor/conf.d/privoxy.conf && \
    echo "stdout_logfile = /var/log/privoxy/privoxy.log" >> /etc/supervisor/conf.d/privoxy.conf && \
    echo "stderr_logfile = /var/log/privoxy/privoxy.err" >> /etc/supervisor/conf.d/privoxy.conf

ADD firefox/syspref.js       /etc/firefox/syspref.js

ADD chromium-browser/default /etc/chromium-browser/default
