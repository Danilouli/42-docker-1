FROM ubuntu:latest
LABEL maintainer gitlabdead@eheh.cq

RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install sudo -y
RUN sudo apt-get install curl -y
RUN sudo apt-get install openssh-server -y
RUN sudo apt-get install ca-certificates -y
RUN sudo apt-get install postfix -y
RUN curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
RUN sudo apt-get install gitlab-ce
RUN sudo gitlab-rake gitlab:setup RAILS_ENV=production
RUN sudo gitlab-ctl reconfigure
CMD sudo gitlab-ctl restart
