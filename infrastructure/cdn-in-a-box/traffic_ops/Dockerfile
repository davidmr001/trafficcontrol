# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

############################################################
# Dockerfile to build Traffic Ops container images
# Based on CentOS 7.2
############################################################


FROM centos:7

RUN yum install -y https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-redhat96-9.6-3.noarch.rpm
RUN yum install -y \
	cpanminus \
	epel-release \
	nmap-ncat \
	openssl \
	perl \
	perl-Crypt-ScryptKDF \
	perl-Test-CPAN-Meta && \
	yum clean all

RUN cpanm Carton
RUN yum install -y perl-DBIx-Connector

# Override TRAFFIC_OPS_RPM arg to use a different one using --build-arg TRAFFIC_OPS_RPM=...  Can be local file or http://...
ARG TRAFFIC_OPS_RPM=traffic_ops.rpm
ADD $TRAFFIC_OPS_RPM /
RUN yum install -y /$(basename $TRAFFIC_OPS_RPM)

# once installed, remove rpm to lower image size
RUN rm /$(basename $TRAFFIC_OPS_RPM) && \
    yum clean all

RUN POSTGRES_HOME=/usr/pgsql-9.6 cd /opt/traffic_ops/app && carton && rm -rf $HOME/.cpan*

RUN /opt/traffic_ops/install/bin/install_go.sh && \
    /opt/traffic_ops/install/bin/install_goose.sh

RUN export PERL5LIB=/opt/traffic_ops/app/local/lib/perl5/:/opt/traffic_ops/install/lib/ \
	&& export TERM=xterm \
	&& export USER=root \
	&& /opt/traffic_ops/install/bin/download_web_deps -i

# \todo add Drive Letters to postinstall input
# RUN sed -i -- 's/"value": "b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y"/"value": "0"/g' /opt/traffic_ops/install/data/profiles/profile.trafficserver_edge.traffic_ops
# RUN sed -i -- 's/"value": "b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y"/"value": "0"/g' /opt/traffic_ops/install/data/profiles/profile.trafficserver_mid.traffic_ops

RUN echo "{\"user\": \"riakuser\",\"password\": \"$TRAFFIC_VAULT_PASS\"}" > /opt/traffic_ops/app/conf/production/riak.conf

EXPOSE 443
WORKDIR /opt/traffic_ops/app
ENV MOJO_MODE production

ADD adduser.pl /
ADD config.sh /
ADD run.sh /
CMD /run.sh
