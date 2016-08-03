### Set the base image to Fedora
FROM jboss/base-jdk:8

## docker build -t jboss-ivt jboss
## docker run -p 8080:8080 -p 9990:9990 -t jboss-ivt

### File Author / Maintainer
MAINTAINER "Greg Autric" "gautric@redhat.com"

### Set Environment
ENV JBOSS_HOME  /opt/jboss/jboss-eap-7.0
ENV MQM_HOME    /opt/mqm
ENV MQM_URL     http://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/messaging/mqadv/mqadv_dev90_linux_x86-64.tar.gz

### Install EAP 6.3.0
COPY jboss-eap-7.0.0.zip /opt
RUN unzip /opt/jboss-eap-7.0.0.zip  1> /dev/null && echo "Install JBoss instance"

### JBoss configuration files
COPY jboss-overlay.cli  /opt
COPY jboss-ejb3.xml     /opt
COPY jboss-web.xml      /opt

## Configure WMQ
#COPY mqadv_dev90_linux_x86-64.tar.gz     /tmp

## Change user to install some
USER root
RUN cd /tmp && yum install -y wget && wget ${MQM_URL} && tar -zxvf ./*.tar.gz  1> /dev/null && cd MQServer && ./mqlicense.sh -text_only -accept  && rpm -ivh  MQSeriesJava*.rpm MQSeriesRuntime*.rpm
RUN rm -rf /tmp/*

### Create Admin EAP user
USER jboss
RUN $JBOSS_HOME/bin/add-user.sh admin admin123! --silent

## Copy EAR and RAR
RUN cp ${MQM_HOME}/java/lib/jca/wmq.jmsra.ivt.ear ${JBOSS_HOME}/standalone/deployments
RUN cp ${MQM_HOME}/java/lib/jca/wmq.jmsra.rar     ${JBOSS_HOME}/standalone/deployments

### Configure EAP
RUN echo "JAVA_OPTS=\"\$JAVA_OPTS -Djboss.bind.address=0.0.0.0 -Djboss.bind.address.management=0.0.0.0\"" >> $JBOSS_HOME/bin/standalone.conf
RUN ${JBOSS_HOME}/bin/jboss-cli.sh --file=/opt/jboss-overlay.cli
RUN rm -rf ${JBOSS_HOME}/standalone/configuration/standalone_xml_history/current
RUN chown -R jboss:jboss ${JBOSS_HOME}

### Open Ports
EXPOSE 8080 9990

### Start EAP
CMD  ${JBOSS_HOME}/bin/standalone.sh -c standalone-full.xml
