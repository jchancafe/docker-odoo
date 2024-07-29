FROM ubuntu:18.04
LABEL MAINTAINER="Johnny Chancafe <jchancafer@gmail.com>"
LABEL NAME=odoo-for-developer
LABEL VERSION=12.0

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8
ENV ODOO_VERSION 12.0

# Use backports to avoid install some libs with pip
RUN apt-get update && \
    apt install -y --no-install-recommends \
    python3-pip \
    curl \
    python3-dev \
    python3-venv \
    python3-wheel \
    libxml2-dev \
    libpq-dev \
    libjpeg8-dev \
    liblcms2-dev \
    libxslt1-dev \
    zlib1g-dev \
    libsasl2-dev \
    libldap2-dev \
    build-essential \
    libssl-dev \
    libffi-dev  \
    libmysqlclient-dev  \
    libjpeg-dev  \
    libblas-dev  \
    libatlas-base-dev  \
    libssl1.1 \
    pkg-config  \
    libxmlsec1-dev \
    libxmlsec1-openssl \
    xmlsec1 \
    git \
    libxmlsec1-dev \
    python3-pysimplesoap
RUN pip3 install --upgrade pip

RUN curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.bionic_amd64.deb \
    && apt install -y --no-install-recommends ./wkhtmltox.deb 

RUN wkhtmltopdf --version

RUN pip install "setuptools==58.0.0"

# install lib python for Facturacion electronica.
RUN pip3 install beautifulsoup4==4.7.1 \
                olefile==0.46 \
                pbr==5.1.3 \
                pdf417gen==0.7.0 \
                py-dateutil==2.2 \
                PySimpleSOAP==1.16.2 \
                python-barcode==0.9.0 \
                python-stdnum==1.11 \
                PyYAML==3.12 \
                query-string==2018.11.20 \
                simplejson==3.16.0 \
                six==1.12.0 \
                soupsieve==1.9 \
                urllib3==1.24.1 \
                xmlsec==1.3.12 \
                bravado-core

# Creando ambiente para Odoo
# - Creando usuario y directorios
RUN useradd -ms /bin/bash odoo12 \
    && mkdir /etc/odoo /opt/odoo12 /opt/odoo12/scripts /opt/odoo12/devel-addons \
    /opt/odoo12/data /opt/odoo12/custom_addons \
    /opt/odoo12/.vscode /opt/odoo12/.vscode-server 
    
WORKDIR /opt/odoo12
# Install Odoo and dependencies from source and check out specific revision
RUN git clone --branch=12.0 --depth=1000 https://github.com/odoo/odoo.git base

RUN chown -R odoo12:odoo12 /etc/odoo /opt/odoo12

COPY ./extras/requirements-fe.txt /opt/odoo12/
#COPY ./extras/requirements.txt /opt/odoo12/

# Install Odoo python package requirements
USER root
RUN pip3 install pip --upgrade
RUN pip3 install --no-cache-dir -r /opt/odoo12/base/requirements.txt
RUN pip3 install --no-cache-dir -r /opt/odoo12/requirements-fe.txt

# Copy entrypoint script and Odoo configuration file
COPY ./entrypoint.sh /
COPY ./odoo.conf /etc/odoo/

# Mount /var/lib/odoo to allow restoring filestore and /mnt/devel-addons for users addons
VOLUME ["/opt/odoo12/data", "/opt/odoo12/.vscode", "/opt/odoo12/base", "/opt/odoo12/devel-addons"]


# Expose Odoo services
EXPOSE 8069 8071 8072

# Set the default config file
ENV ODOO_RC /etc/odoo/odoo.conf

COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER odoo12

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo12"]
