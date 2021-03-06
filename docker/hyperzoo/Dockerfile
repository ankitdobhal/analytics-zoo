FROM ubuntu:18.04

MAINTAINER The Analytics-Zoo Authors https://github.com/intel-analytics/analytics-zoo

ARG ANALYTICS_ZOO_VERSION=0.10.0-SNAPSHOT
ARG BIGDL_VERSION=0.12.1
ARG SPARK_VERSION=2.4.3
ARG FLINK_VERSION=1.10.0
ARG JDK_VERSION=8u192
ARG JDK_URL=your_jdk_url

ENV FLINK_HOME                          /opt/flink-${FLINK_VERSION}
ENV ANALYTICS_ZOO_VERSION               ${ANALYTICS_ZOO_VERSION}
ENV SPARK_VERSION                       ${SPARK_VERSION}
ENV BIGDL_VERSION                       ${BIGDL_VERSION}

ENV OMP_NUM_THREADS                     4
ENV NOTEBOOK_PORT                       12345
ENV NOTEBOOK_TOKEN                      1234qwer

ENV RUNTIME_SPARK_MASTER                local[4]
ENV RUNTIME_K8S_SERVICE_ACCOUNT         spark
ENV RUNTIME_K8S_SPARK_IMAGE             intelanalytics/hyper-zoo:${ANALYTICS_ZOO_VERSION}-${SPARK_VERSION}
ENV RUNTIME_DRIVER_HOST                 localhost
ENV RUNTIME_DRIVER_PORT                 54321
ENV RUNTIME_EXECUTOR_CORES              4
ENV RUNTIME_EXECUTOR_MEMORY             20g
ENV RUNTIME_EXECUTOR_INSTANCES          1
ENV RUNTIME_TOTAL_EXECUTOR_CORES        4
ENV RUNTIME_DRIVER_CORES                4
ENV RUNTIME_DRIVER_MEMORY               10g
ENV RUNTIME_PERSISTENT_VOLUME_CLAIM     myvolumeclaim

ENV SPARK_HOME                          /opt/spark
ENV HADOOP_CONF_DIR                     /opt/hadoop-conf
ENV ANALYTICS_ZOO_HOME                  /opt/analytics-zoo-${ANALYTICS_ZOO_VERSION}
ENV BIGDL_CLASSPATH                     ${ANALYTICS_ZOO_HOME}/lib/analytics-zoo-bigdl_${BIGDL_VERSION}-spark_${SPARK_VERSION}-${ANALYTICS_ZOO_VERSION}-jar-with-dependencies.jar
ENV JAVA_HOME                           /opt/jdk
ENV REDIS_HOME                          /opt/redis-5.0.5
ENV CS_HOME                             /opt/work/cluster-serving
ENV PYTHONPATH                          ${ANALYTICS_ZOO_HOME}/lib/analytics-zoo-bigdl_${BIGDL_VERSION}-spark_${SPARK_VERSION}-${ANALYTICS_ZOO_VERSION}-python-api.zip:${SPARK_HOME}/python/lib/pyspark.zip:${SPARK_HOME}/python/lib/py4j-*.zip:${CS_HOME}/serving-python.zip:/opt/models/research/slim
ENV PATH                                ${ANALYTICS_ZOO_HOME}/bin/cluster-serving:${JAVA_HOME}/bin:/root/miniconda3/bin:${PATH}


RUN apt-get update --fix-missing && \
    apt-get install -y apt-utils vim curl nano wget unzip maven git && \
    apt-get install -y gcc g++ make && \
    apt-get install -y libsm6 libxext6 libxrender-dev && \
    rm /bin/sh && \
    ln -sv /bin/bash /bin/sh && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd

#conda
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-py37_4.8.3-Linux-x86_64.sh && \
    mkdir /root/.conda && \
    bash Miniconda3-py37_4.8.3-Linux-x86_64.sh -b && \
    rm -f Miniconda3-py37_4.8.3-Linux-x86_64.sh && \
#java
    wget $JDK_URL && \
    gunzip jdk-$JDK_VERSION-linux-x64.tar.gz && \
    tar -xf jdk-$JDK_VERSION-linux-x64.tar -C /opt && \
    rm jdk-$JDK_VERSION-linux-x64.tar && \
    mv /opt/jdk* /opt/jdk$JDK_VERSION && \
    ln -s /opt/jdk$JDK_VERSION /opt/jdk && \
#spark
    wget https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop2.7.tgz && \
    tar -zxvf spark-${SPARK_VERSION}-bin-hadoop2.7.tgz && \
    mv spark-${SPARK_VERSION}-bin-hadoop2.7 /opt/spark && \
    rm spark-${SPARK_VERSION}-bin-hadoop2.7.tgz && \
    cp /opt/spark/kubernetes/dockerfiles/spark/entrypoint.sh /opt
ENV TINI_VERSION v0.18.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /sbin/tini
RUN chmod +x /sbin/tini

#python
RUN apt-get install -y python3-minimal && \
    apt-get install -y build-essential python3 python3-setuptools python3-dev python3-pip && \
    pip3 install --upgrade pip && \
    pip install --upgrade setuptools && \
    pip install numpy scipy && \
    pip install pandas==1.0.3 && \
    pip install scikit-learn matplotlib seaborn jupyter jupyterlab wordcloud moviepy requests h5py opencv-python tensorflow==1.15.0 && \
    pip install torch==1.7.1 torchvision==0.8.2  && \
    ln -s /usr/bin/python3 /usr/bin/python && \
    #Fix tornado await process
    pip uninstall -y -q tornado && \
    pip install tornado && \
    python3 -m ipykernel.kernelspec && \
    pip install tensorboard && \
    pip install jep && \
    pip install cloudpickle && \
    pip install opencv-python && \
    pip install pyyaml && \
    pip install redis && \
    pip install keras && \
    pip install mxnet-mkl==1.6.0 && \
    pip install ray==0.8.4 && \
    pip install Pillow==6.1 && \
    pip install psutil aiohttp && \
    pip install py4j && \
    pip install gym gym[atari] && \
    pip install lz4 && \
    pip install ray[rllib] && \
    pip install cmake==3.16.3 && \
    pip install horovod==0.19.2 && \
    git clone https://github.com/tensorflow/models/ /opt/models && \
#redis
    wget http://download.redis.io/releases/redis-5.0.5.tar.gz && \
    tar xzf redis-5.0.5.tar.gz -C /opt && \
    rm redis-5.0.5.tar.gz && \
    cd /opt/redis-5.0.5 && \
    make && \
    echo "bind 0.0.0.0" >> /opt/redis-5.0.5/redis.conf && \
    cd ..

#zoo
ADD ./download-analytics-zoo.sh /opt
ADD ./start-notebook-spark.sh /opt
ADD ./start-notebook-k8s.sh /opt
RUN chmod a+x /opt/download-analytics-zoo.sh && \
    chmod a+x /opt/start-notebook-spark.sh && \
    chmod a+x /opt/start-notebook-k8s.sh && \
    mkdir -p /opt/analytics-zoo-examples/scala && \
    mkdir -p /opt/analytics-zoo-examples/python
RUN /opt/download-analytics-zoo.sh && \
    rm analytics-zoo-bigdl*.zip && \
    unzip $ANALYTICS_ZOO_HOME/lib/*.zip 'zoo/examples/*' -d /opt/analytics-zoo-examples/python && \
    mv /opt/analytics-zoo-examples/python/zoo/examples/* /opt/analytics-zoo-examples/python && \
    rm -rf /opt/analytics-zoo-examples/python/zoo/examples

#cluster-serving
#flink
RUN wget https://archive.apache.org/dist/flink/flink-1.10.0/flink-${FLINK_VERSION}-bin-scala_2.11.tgz && \
    tar xzf flink-*.tgz  -C /opt && \
    rm flink-*.tgz


ADD *.sh /opt/work/scripts/
RUN chmod a+x /opt/work/scripts/*.*

RUN /opt/work/scripts/download-cluster-serving-all-zip.sh
RUN cd cluster-serving && \
    ./cluster-serving-setup.sh

ADD resources/ /opt/work/cluster-serving/resources/
ADD *.py /opt/work/cluster-serving/
ADD perf/ /opt/work/perf/
ADD freeze_checkpoint.py /opt/work

RUN chmod a+x /opt/work/cluster-serving/*


RUN sed -i "s/jobmanager\.heap\.size:.*/jobmanager\.heap\.size: 8g/g" $FLINK_HOME/conf/flink-conf.yaml && \
    sed -i "s/taskmanager\.memory\.process\.size:.*/taskmanager\.memory\.process\.size: 8g/g" $FLINK_HOME/conf/flink-conf.yaml


WORKDIR /opt/spark/work-dir

ENTRYPOINT [ "/opt/entrypoint.sh" ]
