#############################################
# oracle 11g image
# oracle install account：oracle:oracle
# oracle base: /home/oracle/server/
# oracle home: /home/oracle/server/product/11.2/db/
# oracle sid : orcl
# oracle password: oracle
#############################################
FROM centos:6.8
MAINTAINER zhourimeng@gmail.com
RUN yum install -y compat-libstdc++-33 
    elfutils-libelf-devel \
    elfutils-libelf-devel-static \
    ksh \
    libaio \
    libaio-devel \
    libstdc++-devel \
    make \
    numactl-devel \
    sysstat \
    gcc-c++ \
    && yum install -y libXext \
    && yum clean all
RUN groupadd -g 601 dba \
    && groupadd -g 602 oinstall \
    && useradd -u 601 -g 602 -G 601 -m oracle \
    && chmod 750 /home/oracle \
    && echo "oracle:oracle" | chpasswd
RUN echo    "fs.aio-max-nr = 1048576" >>/etc/sysctl.conf \
    && echo "fs.file-max = 6815744" >>/etc/sysctl.conf \
    && echo "kernel.shmmni = 4096" >>/etc/sysctl.conf \
    && echo "kernel.sem = 250 32000 100 128" >>/etc/sysctl.conf \
    && echo "net.ipv4.ip_local_port_range = 9000 65500" >>/etc/sysctl.conf \
    && echo "net.core.rmem_default = 262144" >>/etc/sysctl.conf \
    && echo "net.core.rmem_max = 4194304" >>/etc/sysctl.conf \
    && echo "net.core.wmem_default = 262144" >>/etc/sysctl.conf \
    && echo "net.core.wmem_max = 1048576" >>/etc/sysctl.conf
RUN echo    "oracle              soft    nproc   2047" >>/etc/security/limits.conf \
    && echo "oracle              hard    nproc   16384" >>/etc/security/limits.conf \
    && echo "oracle              soft    nofile  1024" >>/etc/security/limits.conf \
    && echo "oracle              hard    nofile  65536" >>/etc/security/limits.conf \
    && echo "oracle              soft    stack   10240" >> /etc/security/limits.conf
RUN echo    'ORACLE_SID=orcl' >>/home/oracle/.bash_profile \
    && echo 'ORACLE_BASE=$HOME/server' >>/home/oracle/.bash_profile \
    && echo 'ORACLE_HOME=$ORACLE_BASE/product/11.2/db' >>/home/oracle/.bash_profile \
    && echo 'PATH=$ORACLE_HOME/bin:$PATH' >>/home/oracle/.bash_profile \
    && echo 'LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH' >>/home/oracle/.bash_profile \
    && echo 'export ORACLE_SID ORACLE_BASE ORACLE_HOME PATH LD_LIBRARY_PATH' >>/home/oracle/.bash_profile
RUN sed -i '/\/dev\/shm/s/defaults/defaults,size=1g/' /etc/fstab
COPY database /mnt/database
COPY db_install.rsp /mnt/
COPY netca.rsp /mnt/
RUN chown -R oracle:oinstall /mnt && chown -R oracle:oinstall /home/oracle
RUN su - oracle -c "/mnt/database/runInstaller -silent -responseFile /mnt/db_install.rsp -ignorePrereq -waitforcompletion"
RUN /home/oracle/server/oraInventory/orainstRoot.sh \
    && /home/oracle/server/product/11.2/db/root.sh

USER oracle
RUN . ~/.bash_profile \
    && netca -silent -responsefile /mnt/netca.rsp
    && dbca -silent -createDatabase -templateName /home/oracle/server/product/11.2/db/assistants/dbca/templates/General_Purpose.dbc \
        -gdbname orcl -sid orcl -sysPassword sys -systemPassword sys \
        -characterSet ZHS16GBK -nationalCharacterSet AL16UTF16
USER root
RUN rm -rdf /mnt/database

CMD /bin/bash
