FROM tomcat:latest

# Restore Tomcat's default applications, including the Manager application.
RUN cp -R /usr/local/tomcat/webapps.dist/. /usr/local/tomcat/webapps/

# Deploy the application WAR.
COPY ./*.war /usr/local/tomcat/webapps/

# Configure Tomcat Manager access.
COPY manager-context.xml /usr/local/tomcat/webapps/manager/META-INF/context.xml

# Use the Tomcat credentials already configured on Dockerhost.
COPY tomcat-users.xml /usr/local/tomcat/conf/tomcat-users.xml