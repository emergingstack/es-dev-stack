FROM tflowgpu:latest
# Spark dependencies
ENV APACHE_SPARK_VERSION 1.6.1
RUN apt-get -y update && \
    apt-get install -y --no-install-recommends openjdk-7-jre-headless && \
    apt-get clean
RUN apt-get install -y wget
RUN wget -qO - http://d3kbcqa49mib13.cloudfront.net/spark-${APACHE_SPARK_VERSION}-bin-hadoop2.4.tgz | tar -xz -C /usr/local/
RUN cd /usr/local && ln -s spark-${APACHE_SPARK_VERSION}-bin-hadoop2.4 spark


# Scala Spark kernel (build and cleanup)
RUN cd /tmp && \
    echo deb http://dl.bintray.com/sbt/debian / > /etc/apt/sources.list.d/sbt.list && \
    apt-get update && \
    git clone https://github.com/ibm-et/spark-kernel.git && \
    apt-get install -yq --force-yes --no-install-recommends sbt && \
    cd spark-kernel && \
    git checkout 3905e47815 && \
    make dist SHELL=/bin/bash && \
    mv dist/spark-kernel /opt/spark-kernel && \
    chmod +x /opt/spark-kernel && \
    rm -rf ~/.ivy2 && \
    rm -rf ~/.sbt && \
    rm -rf /tmp/spark-kernel && \
    apt-get remove -y sbt && \
    apt-get clean

# Spark and Mesos pointers
ENV SPARK_HOME /usr/local/spark
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.8.2.1-src.zip
ENV PYSPARK_SUBMIT_ARGS --master local[8] --driver-memory 8g --executor-memory 8g --driver-class-path /opt/mysql-connector-java-5.1.38-bin.jar pyspark-shell

RUN mkdir -p /usr/local/share/jupyter/kernels/scala
COPY kernel.json /usr/local/share/jupyter/kernels/scala/

WORKDIR /opt
RUN wget http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.38.tar.gz
RUN tar -xzvf mysql-connector-java-5.1.38.tar.gz

RUN cp /opt/mysql-connector-java-5.1.38/mysql-connector-java-5.1.38-bin.jar /opt
WORKDIR /examples

RUN pip install py4j

RUN cp /usr/local/spark/conf/log4j.properties.template /usr/local/spark/conf/log4j.properties
RUN sed -i 's/rootCategory=INFO/rootCategory=WARN/g' /usr/local/spark/conf/log4j.properties
RUN \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  add-apt-repository -y ppa:webupd8team/java && \
  apt-get update && \
  apt-get install -y oracle-java8-installer


RUN apt-get install -y oracle-java8-set-default
