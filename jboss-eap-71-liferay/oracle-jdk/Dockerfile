# 
# Liferay DXP + JBoss EAP 7.1 for Openshift over Oracle JDK
# 
# docker build \
# --build-arg LIFERAY_PACKAGE_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-dxp-digital-enterprise-7.0-sp6-20171010144253003.war" \
# --build-arg LIFERAY_DEPENDENCIES_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-dxp-digital-enterprise-dependencies-7.0-sp6-20171010144253003.zip" \
# --build-arg LIFERAY_LICENSE_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/activation-key-development.xml" \
# --build-arg LIFERAY_PATCHING_TOOL_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/patching-tool-2.0.7.zip" \
# --build-arg LIFERAY_OSGI_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-dxp-digital-enterprise-osgi-7.0-sp6-20171010144253003.zip" \
# --build-arg LIFERAY_TOOLS_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-dxp-digital-enterprise-tools-7.0-sp6-20171010144253003.zip" \
# --build-arg LIFERAY_WORK_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-dxp-digital-enterprise-work-7.0-sp6-20171010144253003.zip" \
# --build-arg LIFERAY_FIX_PACK="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-fix-pack-de-39-7010.zip" \
# -t liferay/dxp-jboss71:oraclejdk . 
#
FROM liferay/jboss-eap-base:71
MAINTAINER Andre Fabbro <andre.fabbro@liferay.com>

ARG LIFERAY_PACKAGE_URL
ARG	LIFERAY_DEPENDENCIES_URL
ARG	LIFERAY_LICENSE_URL
ARG	LIFERAY_PATCHING_TOOL_URL
ARG	LIFERAY_OSGI_URL
ARG	LIFERAY_TOOLS_URL
ARG LIFERAY_WORK_URL
ARG	LIFERAY_FIX_PACK

ENV JAVA_HOME="/usr/java/jdk1.8.0_161" \
	JAVA_VENDOR="oracle" \
	JAVA_VERSION="1.8.0_161-b12"

WORKDIR /tmp

USER root

RUN rm -rf /tmp/* \
	&& chown jboss:root /opt \
	&& touch /opt/.liferay-home \
	&& chown jboss:root /opt/.liferay-home \
	&& rm -rf /deployments/activemq-rar.rar

# Get the Liferay WAR and place in JBoss deployment folder
RUN wget $LIFERAY_PACKAGE_URL \
	&& unzip -q /tmp/*.war -d /deployments/ROOT.war \
	&& touch /deployments/ROOT.war.dodeploy \
    && rm *.war \
	&& chown -R jboss:root /deployments

# Get the Liferay dependencies and put into JBoss modules folder
RUN wget $LIFERAY_DEPENDENCIES_URL \
	&& unzip -q *.zip \
    && mkdir -p /opt/eap/modules/com/liferay/portal/main \
    && mv */*.jar /opt/eap/modules/com/liferay/portal/main/ \
    && touch /opt/eap/modules/com/liferay/portal/main/module.xml \
    && rm -rf * \
	&& chown -R jboss:root /opt/eap/modules/com

# Get the Liferay license
RUN wget $LIFERAY_LICENSE_URL \
	&& mkdir -p /opt/deploy \
	&& mv *.xml /opt/deploy/ \
	&& chown -R jboss:root /opt/deploy

# Install de patching tool
RUN wget $LIFERAY_PATCHING_TOOL_URL \
	&& unzip -q *.zip \
	&& mv patching-tool /opt/ \
	&& rm *.zip \
	&& chown -R jboss:root /opt/patching-tool
	
# Install the OSGi folder
RUN wget $LIFERAY_OSGI_URL \
	&& unzip -q *.zip \
	&& mv */osgi /opt/ \
	&& rm -rf * \
	&& chown -R jboss:root /opt/osgi

# Install Tools
RUN wget $LIFERAY_TOOLS_URL \
	&& unzip -q *.zip \
	&& rm -rf *.zip \
	&& mv liferay-portal-tools-* /opt/tools \
	&& chown -R jboss:root /opt/tools

# Install work folder
RUN wget $LIFERAY_WORK_URL \
	&& mkdir -p /opt/work \
	&& unzip -q *.zip \
	&& rm -rf *.zip \
	&& mv * /opt/work/ \
	&& chown -R jboss:root /opt/work
	
# Get the fix pack
RUN wget $LIFERAY_FIX_PACK \
	&& mv *.zip /opt/patching-tool/patches/ \
	&& chown -R jboss:root /opt/patching-tool
	
# Create datafolder
RUN mkdir -p /opt/data \
	&& chown -R jboss:root /opt/data
	
# Create datafolder
RUN mkdir -p /opt/logs \
	&& chown -R jboss:root /opt/logs
	
# Copy the sunjce provider to the modules
RUN cp $JAVA_HOME/jre/lib/ext/sunjce_provider.jar /opt/eap/modules/com/liferay/portal/main/

# Add the modules descriptor for Liferay Dependencies
RUN echo $'<?xml version="1.0"?> \n\
<module xmlns="urn:jboss:module:1.1" name="com.liferay.portal"> \n\
    <resources> \n\
        <resource-root path="com.liferay.registry.api.jar" /> \n\
        <resource-root path="hsql.jar" /> \n\
        <resource-root path="mariadb.jar" /> \n\
		<resource-root path="portal-kernel.jar" /> \n\
		<resource-root path="portlet.jar" /> \n\
		<resource-root path="sunjce_provider.jar" /> \n\
    </resources> \n\
    <dependencies> \n\
        <module name="javax.api" /> \n\
        <module name="javax.mail.api" /> \n\
        <module name="javax.servlet.api" /> \n\
        <module name="javax.servlet.jsp.api" /> \n\
        <module name="javax.transaction.api" /> \n\
    </dependencies> \n\
</module>' >> /opt/eap/modules/com/liferay/portal/main/module.xml

#
# Instrument the standalone.xml with the specific configurations for the portal
#
RUN sed -i '/<\/extensions>/a\\n    <system-properties>\n\t<property name="org.apache.catalina.connector.URI_ENCODING" value="UTF-8"\/>\n\t<property name="org.apache.catalina.connector.USE_BODY_ENCODING_FOR_QUERY_STRING" value="true"\/>\n    <\/system-properties>\n' /opt/eap/standalone/configuration/standalone.xml \
	&& sed -i '/scan-interval="5000"/c \\t    <deployment-scanner path="deployments" relative-to="jboss.server.base.dir" scan-interval="5000" deployment-timeout="360" runtime-failure-causes-rollback="${jboss.deployment.scanner.rollback.on.failure:false}" />' /opt/eap/standalone/configuration/standalone.xml \
	&& sed -i '/jsp-config/c \\t    <jsp-config development="true" />' /opt/eap/standalone/configuration/standalone.xml \
	&& sed -i '/handler="welcome-content"/d' /opt/eap/standalone/configuration/standalone.xml \
	&& sed -i '0,/<\/security-domains>/! s/<\/security-domains>/\n\t\t<security-domain name="PortalRealm">\n\t\t    <authentication>\n\t\t\t<login-module code="com.liferay.portal.security.jaas.PortalLoginModule" flag="required"\/>\n\t\t    <\/authentication>\n\t\t<\/security-domain>\n\t<\/security-domains>/' /opt/eap/standalone/configuration/standalone.xml

#
# Add com/sun/crypto and other dependencies in sun/jdk/ module
#
RUN sed -i '/<\/paths>/i\\n\t\t<path name="com/sun/crypto" />\n\t\t<path name="com/sun/org/apache/xml/internal/resolver" />\n\t\t<path name="com/sun/org/apache/xml/internal/resolver/tools" />\n' /opt/eap/modules/system/layers/base/sun/jdk/main/module.xml

# Create the default JBoss security policy
RUN echo $'grant { \n\
    permission java.security.AllPermission; \n\
};' > /opt/eap/bin/server.policy \
	&& chown -R jboss:root /opt/eap/bin/server.policy

# Activate the JBoss Security Manager
# RUN sed -i '/# SECMGR="true"/c SECMGR="true"' /opt/eap/bin/standalone.conf

# Insert custom JVM Options, 1.5GB for heap size
RUN sed -i '/JAVA_OPTS="-Xms1303m -Xmx1303m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true"/c \\n   JAVA_OPTS="-Dfile.encoding=UTF-8 -Djava.net.preferIPv4Stack=true -Djava.security.policy=\/opt\/eap\/bin\/server.policy -Djboss.home.dir=\/opt\/eap -Duser.timezone=GMT -Xmx1536m -Xms1536m"' /opt/eap/bin/standalone.conf

# Create a default portal-ext.properties
RUN touch /opt/portal-ext.properties
RUN echo $'admin.email.from.address=test@liferay.com\n\
admin.email.from.name=Test Test\n\
liferay.home=/opt\n\
setup.wizard.enabled=false' >> /opt/portal-ext.properties
RUN chown -R jboss:root /opt/portal-ext.properties

# USER jboss
WORKDIR /opt/eap

# Install the lastest patch
#RUN cd /opt/patching-tool/ \
#    && sh patching-tool.sh auto-discovery \
#    && sh patching-tool.sh install

# Ensure signals are forwarded to the JVM process correctly for graceful shutdown
ENV LAUNCH_JBOSS_IN_BACKGROUND true

# Expose the ports we're interested in
EXPOSE 8080 9990 8009 9999

# Set the default command to run on boot
# This will boot JBoss in the standalone mode and bind to all interface
# CMD ["bash"]
CMD ["/opt/eap/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]