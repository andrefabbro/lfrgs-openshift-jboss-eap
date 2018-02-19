#
# JBoss EAP 7.0 for Openshift with Oracle JDK
# 
# Usage: 
# docker build -t liferay/jboss-eap70-base:latest .
#
FROM registry.access.redhat.com/jboss-eap-7/eap70-openshift
MAINTAINER Andre Fabbro <andre.fabbro@liferay.com>

USER root
WORKDIR /tmp

RUN yum update -y && \
	yum install -y wget augeas telnet && \
	wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-linux-x64.rpm && \
	yum localinstall -y jdk-8u161-linux-x64.rpm && \
	rm -f jdk-8u161-linux-x64.rpm && \
	rm -rf /var/cache/yum
	
RUN alternatives --set java /usr/java/jdk1.8.0_161/jre/bin/java && \
	alternatives --set javac /usr/java/jdk1.8.0_161/bin/javac

# Set the default command to run on boot
# This will boot JBoss in the standalone mode and bind to all interface
CMD ["bash"]
