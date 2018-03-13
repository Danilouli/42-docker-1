FROM ubuntu:latest
LABEL maintainer gitlabdead@eheh.cq

RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install sudo -y

#RUN sudo apt-get install -y vim
#RUN sudo update-alternatives --set editor /usr/bin/vim.basic

RUN sudo apt-get install -y build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libre2-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev logrotate rsync python-docutils pkg-config cmake
RUN sudo apt-get install rubygems -y

# Install Git
RUN sudo apt-get install -y git-core

#RUN sudo apt-get install -y postfix

# Ruby
RUN sudo apt-get remove ruby1.8
RUN mkdir /tmp/ruby
WORKDIR /tmp/ruby
RUN curl --remote-name --progress https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.6.tar.gz
RUN echo '4e6a0f828819e15d274ae58485585fc8b7caace0  ruby-2.3.6.tar.gz' | shasum -c - && tar xzf ruby-2.3.6.tar.gz
WORKDIR ruby-2.3.6
RUN ./configure --disable-install-rdoc
RUN make
RUN sudo make install
RUN sudo gem install bundler --no-ri --no-rdoc

# Go
RUN sudo rm -rf /usr/local/go
RUN curl --remote-name --progress https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz
RUN echo '1862f4c3d3907e59b04a757cfda0ea7aa9ef39274af99a784f5be843c80c6772  go1.8.3.linux-amd64.tar.gz' | shasum -a256 -c - && sudo tar -C /usr/local -xzf go1.8.3.linux-amd64.tar.gz
RUN sudo ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/local/bin/
RUN rm go1.8.3.linux-amd64.tar.gz

# Node
RUN curl --location https://deb.nodesource.com/setup_7.x | sudo bash -
RUN sudo apt-get install -y nodejs
RUN curl --silent --show-error https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
RUN sudo apt-get update
RUN sudo apt-get install yarn

# Git user for Gitlab
RUN sudo adduser --disabled-login --gecos 'GitLab' git

# PostgreSQL Install
RUN sudo apt-get install -y postgresql postgresql-client libpq-dev postgresql-contrib
RUN sudo service postgresql start && \
	sudo -u postgres psql -d template1 -c "CREATE USER git CREATEDB;" && \
	sudo -u postgres psql -d template1 -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;" && \
	sudo -u postgres psql -d template1 -c "CREATE DATABASE gitlabhq_production OWNER git;"

# Redis
RUN sudo apt-get install redis-server -y
RUN sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.orig
RUN sed 's/^port .*/port 0/' /etc/redis/redis.conf.orig | sudo tee /etc/redis/redis.conf
RUN echo 'unixsocket /var/run/redis/redis.sock' | sudo tee -a /etc/redis/redis.conf
RUN echo 'unixsocketperm 770' | sudo tee -a /etc/redis/redis.conf
RUN mkdir /var/run/redis
RUN chown redis:redis /var/run/redis
RUN chmod 755 /var/run/redis
RUN if [ -d /etc/tmpfiles.d ]; then echo 'd  /var/run/redis  0755  redis  redis  10d  -' | sudo tee -a /etc/tmpfiles.d/redis.conf; fi
RUN sudo service redis-server restart
RUN sudo usermod -aG redis git

# GitLab
WORKDIR /home/git
RUN sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b 10-5-stable gitlab
WORKDIR /home/git/gitlab
RUN sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml
#RUN sudo -u git -H editor config/gitlab.yml
RUN sudo -u git -H cp config/secrets.yml.example config/secrets.yml
RUN sudo -u git -H chmod 0600 config/secrets.yml
RUN sudo chown -R git log/
RUN sudo chown -R git tmp/
RUN sudo chmod -R u+rwX,go-w log/
RUN sudo chmod -R u+rwX tmp/
RUN sudo chmod -R u+rwX tmp/pids/
RUN sudo chmod -R u+rwX tmp/sockets/
RUN sudo -u git -H mkdir public/uploads/
RUN sudo chmod 0700 public/uploads
RUN sudo chmod -R u+rwX builds/
RUN sudo chmod -R u+rwX shared/artifacts/
RUN sudo chmod -R ug+rwX shared/pages/
RUN sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb
#RUN nproc && sudo -u git -H editor config/unicorn.rb
RUN sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb
RUN sudo -u git -H git config --global core.autocrlf input
RUN sudo -u git -H git config --global gc.auto 0
RUN sudo -u git -H git config --global repack.writeBitmaps true
RUN sudo -u git -H git config --global receive.advertisePushOptions true
RUN sudo -u git -H cp config/resque.yml.example config/resque.yml
#RUN sudo -u git -H editor config/resque.yml

# Gitlab DB Settings
RUN sudo -u git cp config/database.yml.postgresql config/database.yml
RUN sudo -u git -H chmod o-rwx config/database.yml
RUN sudo -u git -H bundle install --deployment --without development test mysql aws kerberos

# GitLab Shell
#WORKDIR /home/git/gitlab-shell
#RUN git init .
#RUN git remote add origin https://gitlab.com/gitlab-org/gitlab-shell.git
#RUN git pull origin master
#RUN sudo -u git -H git fetch --all --tags && \
#	sudo -u git -H git checkout v$(</home/git/gitlab/GITLAB_SHELL_VERSION)
#RUN ls -la
#RUN chmod 777 /home/git/gitlab-shell/.gitlab_shell_secret
#WORKDIR /home/git/gitlab
RUN sudo -u git -H bundle exec rake "gitlab:shell:install[v6.0.3]" REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production SKIP_STORAGE_VALIDATION=true
RUN sudo -u git -H bundle exec rake "gitlab:workhorse:install[/home/git/gitlab-workhorse]" RAILS_ENV=production
#RUN sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production
RUN sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
RUN sudo update-rc.d gitlab defaults 21
# Gitaly pizza
RUN sudo -u git -H bundle exec rake "gitlab:gitaly:install[/home/git/gitaly]" RAILS_ENV=production
RUN sudo chmod 0700 /home/git/gitlab/tmp/sockets/private
RUN sudo chown git /home/git/gitlab/tmp/sockets/private
# Logrotate
RUN sudo cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab
# Launch
RUN sudo -u git -H bundle exec rake gettext:compile RAILS_ENV=production
RUN sudo -u git -H yarn install --production --pure-lockfile
RUN sudo -u git -H bundle exec rake gitlab:assets:compile RAILS_ENV=production NODE_ENV=production
RUN sudo service gitlab start
#OU RUN sudo /etc/init.d/gitlab restart

# Nginx
RUN sudo apt-get install -y nginx
RUN sudo cp lib/support/nginx/gitlab /etc/nginx/sites-available/gitlab
RUN sudo ln -s /etc/nginx/sites-available/gitlab /etc/nginx/sites-enabled/gitlab

CMD sudo service postgresql restart && sudo service gitlab restart
