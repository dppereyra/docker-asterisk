FROM centos:7
MAINTAINER Dennis Philippe Pereyra Jr. <dppereyra@gmail.com>

ARG PJPROJECT_VERSION=2.5.5
ARG PJPROJECT_FILE=pjproject-$PJPROJECT_VERSION.tar.bz2
ARG PJPROJECT_URL=http://www.pjsip.org/release/$PJPROJECT_VERSION/$PJPROJECT_FILE 

ARG ASTERISK_VERSION=13
ARG ASTERISK_FILE=asterisk-$ASTERISK_VERSION-current.tar.gz
ARG ASTERISK_URL=http://downloads.asterisk.org/pub/telephony/asterisk/$ASTERISK_FILE

RUN yum install -y \
    binutils-devel \
    bzip2 \
    dmidecode \
    epel-release \
    gcc-c++ \
    gtk2-devel \
    jansson-devel \
    kernel-devel \
    libuuid-devel \
    libxml2-devel \
    make \
    ncurses-devel \
    newt-devel \
    openssl-devel \
    sqlite-devel \
    svn \
    wget

RUN mkdir -p /build

WORKDIR /build
RUN wget $PJPROJECT_URL
RUN tar -jxvf $PJPROJECT_FILE
RUN rm $PJPROJECT_FILE
RUN mv pjproject-* pjproject

WORKDIR /build/pjproject
RUN ./configure CFLAGS="-DNDEBUG -DPJ_HAS_IPV6=1" \
    --prefix=/usr \
    --libdir=/usr/lib64 \
    --enable-shared \
    --disable-video \
    --disable-sound \
    --disable-opencore-amr
RUN make dep
RUN make
RUN make install
RUN ldconfig

WORKDIR /build
RUN wget $ASTERISK_URL
RUN tar -zxvf $ASTERISK_FILE
RUN rm $ASTERISK_FILE
RUN mv asterisk-* asterisk


WORKDIR /build/asterisk
RUN ./contrib/scripts/get_mp3_source.sh
RUN ./configure \
    --libdir=/usr/lib64
RUN make menuselect.makeopts
RUN menuselect/menuselect \
    --enable cdr_csv \
    --enable chan_sip \
    --enable res_snmp \
    --enable res_http_websocket \
    menuselect.makeopts

RUN make
RUN make install
RUN make samples

WORKDIR /
RUN sed -i -e 's/# MAXFILES=/MAXFILES=/' /usr/sbin/safe_asterisk
RUN sed -i 's/TTY=9/TTY=/g' /usr/sbin/safe_asterisk

RUN useradd -m asterisk -s /sbin/nologin
RUN chown asterisk:asterisk /var/run/asterisk
RUN chown -R asterisk:asterisk /etc/asterisk/
RUN chown -R asterisk:asterisk /var/{lib,log,spool}/asterisk
RUN chown -R asterisk:asterisk /usr/lib64/asterisk/

VOLUME \
    /etc/asterisk \
    /var/lib/asterisk \
    /var/log/asterisk \
    /var/spool/asterisk

USER asterisk
CMD /usr/sbin/asterisk -f -U asterisk -G asterisk -vvvg -c