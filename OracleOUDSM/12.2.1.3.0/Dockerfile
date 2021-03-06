# ----------------------------------------------------------------------
# Trivadis AG, Infrastructure Managed Services
# Saegereistrasse 29, 8152 Glattbrugg, Switzerland
# ----------------------------------------------------------------------
# Name.......: Dockerfile
# Author.....: Stefan Oehrli (oes) stefan.oehrli@trivadis.com
# Editor.....: Stefan Oehrli
# Date.......: 2018.03.19
# Purpose....: This Dockerfile is to build Oracle Unifid Directory
# Notes......: --
# Reference..: --
# License....: Licensed under the Universal Permissive License v 1.0 as
#              shown at http://oss.oracle.com/licenses/upl.
# ----------------------------------------------------------------------
# Modified...:
# see git revision history for more information on changes/updates
# ----------------------------------------------------------------------

# Pull base image
# ----------------------------------------------------------------------
FROM oracle/serverjre:8

# Maintainer
# ----------------------------------------------------------------------
LABEL maintainer="stefan.oehrli@trivadis.com"

# Arguments for Oracle Installation
ARG ORACLE_ROOT
ARG ORACLE_DATA
ARG ORACLE_BASE
ARG ORAREPO

# just my environment variable for the software repository host
ENV ORAREPO=${ORAREPO:-orarepo}

# Environment variables required for this build 
# Change them carefully and wise!
# -------------------------------------------------------------
# Software stage area, repository, binary packages and patchs
ENV DOWNLOAD="/tmp/download" \
    SOFTWARE="/opt/stage" \
    SOFTWARE_REPO="http://$ORAREPO" \
    DOCKER_SCRIPTS="/opt/docker/bin" \
    ORADBA_INIT="/opt/oradba/bin" \
    OUD_BASE_PKG="p26270957_122130_Generic.zip" \
    FMW_BASE_PKG="p26269885_122130_Generic.zip" \
    OUD_PATCH_PKG="p28569189_122130_Generic.zip" \
    FMW_PATCH_PKG="p28298734_122130_Generic.zip" \
    OUD_OPATCH_PKG="p28186730_139400_Generic.zip"

# scripts to build and run this container
ENV SETUP_INIT="00_setup_oradba_init.sh" \
    SETUP_OS="01_setup_os_oud.sh" \
    SETUP_OUD="10_setup_oudsm_12c.sh" \
    SETUP_OUDBASE="20_setup_oudbase.sh" \
    START_SCRIPT="start_oud_instance.sh" \
    CHECK_SCRIPT="check_oud_instance.sh" \
    ORACLE_ROOT=${ORACLE_ROOT:-/u00} \
    ORACLE_DATA=${ORACLE_DATA:-/u01}

# stuff to run an OUD instans
ENV OUD_INSTANCE=${OUD_INSTANCE:-oud_docker} \
    USER_MEM_ARGS="-Djava.security.egd=file:/dev/./urandom" \
    ORACLE_HOME_NAME="fmw12.2.1.3.0" \
    PORT="${PORT:-7001}" \
    PORT_SSL="${PORT_SSL:-7002}" 

# Use second ENV so that variable get substituted
ENV ORACLE_BASE=${ORACLE_BASE:-$ORACLE_ROOT/app/oracle} \
    OUD_INSTANCE_BASE=${OUD_INSTANCE_BASE:-$ORACLE_DATA/instances}

# same same but different...
# third ENV so that variable get substituted
ENV PATH=${PATH}:"${OUD_INSTANCE_HOME}/OUD/bin:${ORACLE_BASE}/product/${ORACLE_HOME_NAME}/oud/bin:${DOCKER_SCRIPTS}" \
    ORACLE_HOME=${ORACLE_BASE}/product/${ORACLE_HOME_NAME}

# Copy scripts and software
# ----------------------------------------------------------------------
# copy all setup scripts to DOCKER_BIN
COPY scripts/* "${DOCKER_SCRIPTS}/"

# RUN as user root
# ----------------------------------------------------------------------
# get the OraDBA init script to setup the OS and later OUD
RUN GITHUB_URL="https://github.com/oehrlis/oradba_init/raw/master/bin" && \
    curl -Lsf ${GITHUB_URL}/${SETUP_INIT} -o ${DOCKER_SCRIPTS}/${SETUP_INIT} && \
    chmod 755 ${DOCKER_SCRIPTS}/${SETUP_INIT} && \
    ${DOCKER_SCRIPTS}/${SETUP_INIT}

# Setup OS using OraDBA init script
RUN ${ORADBA_INIT}/${SETUP_OS}

# COPY oud software 
COPY *zip* "${SOFTWARE}/"

# RUN as oracle
# Switch to user oracle, oracle software as to be installed with regular user
# ----------------------------------------------------------------------
USER oracle
RUN ${ORADBA_INIT}/${SETUP_OUD}

# get the latest OUD base from GitHub and install it
RUN ${ORADBA_INIT}/${SETUP_OUDBASE}

# Finalize image
# ----------------------------------------------------------------------
# expose the OUDSM http and https ports
EXPOSE ${PORT} ${PORT_SSL}

# run container health check
HEALTHCHECK --interval=1m --start-period=5m \
   CMD "${DOCKER_SCRIPTS}/${CHECK_SCRIPT}" >/dev/null || exit 1

# Oracle data volume for OUD instance and configuration files
VOLUME ["${ORACLE_DATA}"]

# set workding directory
WORKDIR "${ORACLE_BASE}"

# Define default command to start OUD instance
CMD exec "${DOCKER_SCRIPTS}/${START_SCRIPT}"
# --- EOF --------------------------------------------------------------