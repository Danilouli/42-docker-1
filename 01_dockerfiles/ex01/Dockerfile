# Base system is Ubuntu 16.04
FROM   debian:latest

# Set the Teamspeak version to download
ENV TSV=3.0.13.8

# Download and install everything from the repos.
RUN apt-get update && apt-get install bzip2

# Download and install TeamSpeak 3
# Add secondary/backup server as well -- allow users to choose in case of blacklisting.
ADD    http://dl.4players.de/ts/releases/${TSV}/teamspeak3-server_linux_amd64-${TSV}.tar.bz2 ./
#ADD    http://teamspeak.gameserver.gamed.de/ts3/releases/${TSV}/teamspeak3-server_linux_amd64-${TSV}.tar.bz2 ./

RUN    tar jxf teamspeak3-server_linux_amd64-$TSV.tar.bz2 && \
       mv teamspeak3-server_linux_amd64 /opt/teamspeak && \
       rm teamspeak3-server_linux_amd64-$TSV.tar.bz2

# Load in all of our config files.
ADD    https://raw.githubusercontent.com/overshard/docker-teamspeak/master/scripts/start /startTS

#RUN touch /startTS
#RUN echo "export LD_LIBRARY_PATH=/opt/teamspeak" > startTS
#RUN echo "cd /opt/teamspeak" >> startTS
#RUN echo "./ts3server logpath=data/logs" >> startTS

# Fix all permissions
RUN    chmod +x /startTS

# /start runs it.
EXPOSE 9987/udp
EXPOSE 30033
EXPOSE 10011

RUN    useradd teamspeak && mkdir /data
CMD    ["/startTS"]

