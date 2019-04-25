##
# geographica/gdal2
#
# This creates an Ubuntu derived base image that installs GDAL 2.
#
# Ubuntu 18.04 Bionic Beaver
FROM ubuntu:bionic

MAINTAINER Cayetano Benavent <cayetano.benavent@geographica.gs>

ENV ROOTDIR /usr/local/
ENV GDAL_VERSION 2.4.1
ENV OPENJPEG_VERSION 2.3.0

# Load assets
WORKDIR $ROOTDIR/

ADD http://download.osgeo.org/gdal/${GDAL_VERSION}/gdal-${GDAL_VERSION}.tar.gz $ROOTDIR/src/
ADD https://github.com/uclouvain/openjpeg/archive/v${OPENJPEG_VERSION}.tar.gz $ROOTDIR/src/openjpeg-${OPENJPEG_VERSION}.tar.gz

# Install basic dependencies
RUN apt-get update -y && apt-get install -y \
    software-properties-common \
    build-essential \
    python-dev \
    python3-dev \
    python-numpy \
    python3-numpy \
    libspatialite-dev \
    sqlite3 \
    libpq-dev \
    libcurl4-gnutls-dev \
    libproj-dev \
    libxml2-dev \
    libgeos-dev \
    libnetcdf-dev \
    libpoppler-dev \
    libspatialite-dev \
    libhdf4-alt-dev \
    libhdf5-serial-dev \
    bash-completion \
    cmake

# Compile and install OpenJPEG
RUN cd src && tar -xvf openjpeg-${OPENJPEG_VERSION}.tar.gz && cd openjpeg-${OPENJPEG_VERSION}/ \
    && mkdir build && cd build \
    && cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$ROOTDIR \
    && make && make install && make clean \
    && cd $ROOTDIR && rm -Rf src/openjpeg*


# NB: By using this Docker image you're accepting the corresponding license:
# https://www.hexagongeospatial.com/technical-documents/erdas-ecwjp2-sdk-desktop-read-only-redistributable-license-agreement
ADD https://go.hexagongeospatial.com/ERDASECWJP2SDKv54Update1forLinux /erdas-ecw-sdk-5.4.0-update1-linux.zip
# sha256: 87d842ae25194237d352bf5947e13e2e8d1214631aaea78264a78d17cba88c9c
RUN unzip /erdas-ecw-sdk-5.4.0-update1-linux.zip
# ADD ERDAS_ECWJP2_SDK-5.4.0.bin /
RUN chmod +x /ERDAS_ECWJP2_SDK-5.4.0.bin \
  && sed -i "/LICENSE=n/{s/=n/=yes/;n;d}" /ERDAS_ECWJP2_SDK-5.4.0.bin \
  && echo 1 | /ERDAS_ECWJP2_SDK-5.4.0.bin

RUN \
  cp -r /hexagon/ERDAS-ECW_JPEG_2000_SDK-5.4.0/Desktop_Read-Only $ROOTDIR/hexagon \
  && rm -r $ROOTDIR/hexagon/lib/x64 \
  && mv $ROOTDIR/hexagon/lib/newabi/x64 $ROOTDIR/hexagon/lib/x64 \
  && cp $ROOTDIR/hexagon/lib/x64/release/libNCSEcw* $ROOTDIR/lib \
  && ldconfig $ROOTDIR/hexagon


# Compile and install GDAL
RUN cd src && tar -xvf gdal-${GDAL_VERSION}.tar.gz && cd gdal-${GDAL_VERSION} \
    && ./configure --with-python --with-spatialite --with-pg --with-curl --with-openjpeg --with-ecw=$ROOTDIR/hexagon \
    && make -j $(nproc) && make install && ldconfig \
    && apt-get update -y \
    && cd $ROOTDIR && cd src/gdal-${GDAL_VERSION}/swig/python \
    && python3 setup.py build \
    && python3 setup.py install \
    && apt-get remove -y --purge build-essential \
      python-dev \
      python3-dev \
    && cd $ROOTDIR && rm -Rf src/gdal*

# Output version and capabilities by default.
CMD gdalinfo --version && gdalinfo --formats && ogrinfo --formats
