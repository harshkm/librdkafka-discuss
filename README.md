# librdkafka-discuss

Librdkafka installation problem [No provider for SASL mechanism GSSAPI] #3091
Closed
Closed
Librdkafka installation problem [No provider for SASL mechanism GSSAPI]
#3091
@AdrianaTheo
Description
AdrianaTheo
opened on Sep 29, 2020
Hello,

I am working on a Kafka consumer written in python using the confluent_kafka v1.5.0 library, now I have to apply Kerberos encryption on it, and I need to install the Librdkafka library (in Ubuntu 20.04).

I tried the below ways to make it but without success.

I Followed the instructions here and then apt install librdkafka-dev but I am getting the following error
The following packages have unmet dependencies:
 librdkafka-dev : Depends: librdkafka1 (= 1.5.0~1confluent6.0.0-1) but it is not going to be installed
                  Depends: librdkafka++1 (= 1.5.0~1confluent6.0.0-1) but it is not going to be installed
Tried to install the unmet dependencies but I found that libssl1.0.0 not being available anymore on Ubuntu >= 18

I installed the librdkafka via Homebrew, but it isn't visible of my project so I am getting the below error:
KafkaError{code=_INVALID_ARG,val=-186,str="Failed to create consumer: No provider for SASL mechanism GSSAPI: recompile librdkafka with libsasl2 or openssl support. Current build options: PLAIN SASL_SCRAM OAUTHBEARER"}

The only thing that I didn't try so far is to install the library via git repo because I don't know how (It would be nice if you could explain me the way)

Thank you in advance.

Activity
edenhill
edenhill commented on Sep 29, 2020
edenhill
on Sep 29, 2020
Contributor
If you install librdkafka separately you will also need to install the python client from source and not through binary wheels since the binary wheels dont contain GSSAPI support (except for on OSX).

edenhill
edenhill commented on Sep 29, 2020
edenhill
on Sep 29, 2020
Contributor
As for the Ubuntu libssl dependency: that's a known issue and we're working on providing Debian packages for newer deb/ubuntu versions soon. In the meantime you'll have to build librdkafka from source on newer Debians.

AdrianaTheo
AdrianaTheo commented on Sep 30, 2020
AdrianaTheo
on Sep 30, 2020 · edited by AdrianaTheo
Author
Thanks for the help, at the end, I cloned the repository of the library and I built for the project.
the problem was the wrong order of library installations. I am attaching here my docker file which I use and works.
Maybe someone finds it helpful. (for me took many weeks to figured out).

FROM python:3.7-slim
WORKDIR /opt/
RUN apt-get update
RUN apt-get -y install libsasl2-dev
RUN apt-get -y install libssl-dev
RUN apt-get -y install git
RUN apt-get -y install python3-pip
RUN git clone https://github.com/edenhill/librdkafka
WORKDIR /opt/librdkafka/
RUN ./configure
RUN make
RUN make install
RUN ldconfig
WORKDIR /opt/
RUN git clone https://github.com/confluentinc/confluent-kafka-python
WORKDIR /tmp/confluent-kafka-python
RUN pip3 install --no-binary :all: confluent-kafka
WORKDIR /opt/
COPY consumer/requirements.txt .
RUN pip3 install -r requirements.txt
COPY consumer/*.py ./
RUN apt-get -y remove git
RUN apt-get -y autoremove
CMD [ "python3", "./consumer.py" ]
AdrianaTheo
closed this as completedon Sep 30, 2020
q26646
q26646 commented on Jan 5, 2021
q26646
on Jan 5, 2021
@edenhill - is this supported now on 20.04? I was able to successfully install librdkafka-dev for ubuntu18.04 but wanted to check when 20.04 will be available, if not already.
