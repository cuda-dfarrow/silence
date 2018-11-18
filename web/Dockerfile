FROM debian
MAINTAINER farrow.daveb@gmail.com

RUN apt-get update && apt-get install -y \
	apache2 \
	apache2-dev \
	curl \
	git \
	libapache2-mod-perl2 \
	default-libmysqlclient-dev \
	mysql-client \
	mysql-server \
	netcat \
	sudo

# install required perl modules from cpan
RUN cpan install \
	CPAN \
	DBI \
	CGI \
	'CGI::Session' \
	'DBD::mysql' \
	'Data::GUID' \
	'Text::Caml'

RUN set -ex \
	# configure apache
	ln -s -f /etc/apache2/mods-available/perl.load /etc/apache2/mods-enabled/perl.load \
	&& rm -f /etc/apache2/sites-enabled/*.conf \
	&& ln -s -f /etc/apache2/sites-available/silence.conf /etc/apache2/sites-enabled/silence.conf \
	&& rm -rf /var/www/html \
	# add the users
	&& useradd -m -s /bin/bash git \
	&& echo "git:git" | chpasswd \
	&& useradd -m -s /bin/bash psimon \
	&& echo "psimon:kathy" | chpasswd \
	&& useradd -m -s /bin/bash agarfunkel \
	&& echo "agarfunkel:JaMes1990" | chpasswd \
	&& echo "agarfunkel ALL=(ALL) NOPASSWD: /bin/cp" >> /etc/sudoers \
	&& echo "done setting up users"

# copy the application source over and configure the webserver
COPY www /var/www/html/
COPY files/silence.conf /etc/apache2/sites-available

USER git
RUN set -ex \
	# set up the git central repo
	&& mkdir /home/git/.ssh \
	&& chmod 0700 /home/git/.ssh \
	&& mkdir -p /home/git/silence-webapp.git \
	&& cd /home/git/silence-webapp.git \
	&& git init --bare \
	&& echo "done setting up git"

USER root

COPY --chown=psimon:psimon files/psimon.history /home/psimon/.bash_history
COPY --chown=agarfunkel:agarfunkel files/agarfunkel.history /home/agarfunkel/.bash_history
COPY --chown=psimon:psimon files/flag1.txt /home/psimon/
COPY --chown=agarfunkel:agarfunkel files/flag2.txt /home/agarfunkel/
COPY --chown=root:root files/flag3.txt /root/
COPY --chown=root:root files/flag_zHHkyfZJUWnx.txt /var/www/html/
COPY --chown=psimon:psimon files/id_rsa /home/psimon/.ssh/
COPY --chown=git:git files/id_rsa.pub /home/git/.ssh/authorized_keys
COPY --chown=psimon:psimon www/ /home/psimon/silence-webapp/
COPY --chown=psimon:psimon files/README.md /home/psimon/silence-webapp/
COPY --chown=psimon:psimon files/setup.sql /home/psimon/silence-webapp/

# set up the silence web app dir for root and check it into git
# this is a janky hack because the ssh server needs to be running
# and i can't figure out how to get thegit commands run as 
# psimon in the same layer as the ssh server start
USER psimon
RUN set -ex \
	&& git config --global user.name "Paul Simon" \
	&& git config --global user.email "psimon@bnsec.com" \
	&& cd /home/psimon/silence-webapp \
	&& git init \
	&& git add . \
	&& git status \
	&& git commit -m "initial commit" \
	&& git remote add origin git@localhost:/home/git/silence-webapp.git \
	&& echo "done setting up psimon's git env"

# set up ssh and check in the code for psimon
USER root
RUN set -ex \
	&& apt-get install openssh-server -y \
	&& service ssh start \
	&& mkdir /root/.ssh \
	&& ssh-keyscan -H localhost > /root/.ssh/known_hosts \
	&& echo "done setting up sshd" \
	&& cd /home/psimon/silence-webapp \
	&& export GIT_SSH_COMMAND='ssh -i /home/psimon/.ssh/id_rsa' \
	&& git push origin master \
	&& rm -rf /home/psimon/silence-webapp \
	&& echo "done pushing code"

EXPOSE 80
#CMD ["/usr/sbin/apache2ctl", "-DFOREGROUND"]
ENTRYPOINT service ssh start && /usr/sbin/apache2ctl -DFOREGROUND
