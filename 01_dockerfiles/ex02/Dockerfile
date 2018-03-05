FROM ruby:2.3

ENV DEBIAN_FRONTEND=noninteractive \
  NODE_VERSION=6.9.1

RUN apt-get update
RUN apt-get install -y postgresql
RUN curl -sSL "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" | tar xfJ - -C /usr/local --strip-components=1 && \
  npm install npm -g
RUN touch ./Gemfile
RUN touch ./Gemfile.lock
RUN gem install rails
RUN gem install bundler
RUN bundle install

ONBUILD COPY . /opt/app/
ONBUILD WORKDIR /opt/app
ONBUILD RUN touch Gemfile
ONBUILD RUN touch Gemfile.lock
ONBUILD RUN gem install bundler
ONBUILD RUN bundle install
ONBUILD RUN rails new /opt/app
ONBUILD RUN rails db:seed
ONBUILD RUN rails db:migrate
ONBUILD EXPOSE 3000
