# Liferay DXP + JBoss 7.0 for Openshift + OpenJDK

#### Usage: 

```
docker build \
 --build-arg LIFERAY_PACKAGE_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-dxp-digital-enterprise-7.0-sp6-20171010144253003.war" \
 --build-arg LIFERAY_DEPENDENCIES_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-dxp-digital-enterprise-dependencies-7.0-sp6-20171010144253003.zip" \
 --build-arg LIFERAY_LICENSE_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/activation-key-development.xml" \
 --build-arg LIFERAY_PATCHING_TOOL_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/patching-tool-2.0.7.zip" \
 --build-arg LIFERAY_OSGI_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-dxp-digital-enterprise-osgi-7.0-sp6-20171010144253003.zip" \
 --build-arg LIFERAY_TOOLS_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-dxp-digital-enterprise-tools-7.0-sp6-20171010144253003.zip" \
 --build-arg LIFERAY_WORK_URL="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-dxp-digital-enterprise-work-7.0-sp6-20171010144253003.zip" \
 --build-arg LIFERAY_FIX_PACK="http://<<YOUR_PACKAGE_REPOSITORY>>/liferay-fix-pack-de-39-7010.zip" \
 -t liferay/dxp-jboss70:openjdk . 
```

Replace **<<YOUR_PACKAGE_REPOSITORY>>** for the repository where you put the installation binaries.

Then: 

```
docker run --name=dxp liferay/dxp-jboss70:openjdk
```