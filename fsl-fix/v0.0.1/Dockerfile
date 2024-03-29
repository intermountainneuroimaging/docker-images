## Emacs, make this -*- mode: sh; -*-
## Dockerfile from rocker/r-ubuntu Github ##

FROM ubuntu:bionic

LABEL org.label-schema.license="GPL-2.0" \
      org.label-schema.vcs-url="https://github.com/rocker-org/r-ubuntu" \
      org.label-schema.vendor="Rocker Project" \
      maintainer="Dirk Eddelbuettel <edd@debian.org>"

## Set a default user. Available via runtime flag `--user docker` 
## Add user to 'staff' group, granting them write privileges to /usr/local/lib/R/site.library
## User should also have & own a home directory (for rstudio or linked volumes to work properly). 
RUN useradd docker \
	&& mkdir /home/docker \
	&& chown docker:docker /home/docker \
	&& addgroup docker staff

RUN apt-get update \
	&& apt-get install -y --no-install-recommends \
		software-properties-common \
                dirmngr \
                ed \
		less \
		locales \
		vim-tiny \
		wget \
		ca-certificates \
        && add-apt-repository --enable-source --yes "ppa:marutter/rrutter4.0" \
        && add-apt-repository --enable-source --yes "ppa:c2d4u.team/c2d4u4.0+"

## Configure default locale, see https://github.com/rocker-org/rocker/issues/19
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen \
	&& locale-gen en_US.utf8 \
	&& /usr/sbin/update-locale LANG=en_US.UTF-8

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8

## This was not needed before but we need it now
ENV DEBIAN_FRONTEND noninteractive

## Otherwise timedatectl will get called which leads to 'no systemd' inside Docker
ENV TZ UTC

# Now install R and littler, and create a link for littler in /usr/local/bin
# Default CRAN repo is now set by R itself, and littler knows about it too
# r-cran-docopt is not currently in c2d4u so we install from source
RUN apt-get update \
        && apt-get install -y --no-install-recommends \
                 littler \
 		 r-base \
 		 r-base-dev \
 		 r-recommended \
  	&& ln -s /usr/lib/R/site-library/littler/examples/install.r /usr/local/bin/install.r \
 	&& ln -s /usr/lib/R/site-library/littler/examples/install2.r /usr/local/bin/install2.r \
 	&& ln -s /usr/lib/R/site-library/littler/examples/installGithub.r /usr/local/bin/installGithub.r \
 	&& ln -s /usr/lib/R/site-library/littler/examples/testInstalled.r /usr/local/bin/testInstalled.r \
 	&& install.r docopt \
 	&& rm -rf /tmp/downloaded_packages/ /tmp/*.rds \
 	&& rm -rf /var/lib/apt/lists/*

CMD ["bash"]


########################################
# R packages

RUN apt update \
    &&  apt-get install -y --no-install-recommends openssl libssl-dev \
    &&  apt install -y --no-install-recommends build-essential libcurl4-gnutls-dev libxml2-dev libssl-dev \
    && Rscript -e 'install.packages("devtools")' \
    && Rscript -e 'chooseCRANmirror(ind=72)' \
    && Rscript -e 'devtools::install_version("kernlab", version="0.9-24")' \
    && Rscript -e 'devtools::install_version("ROCR", version="1.0-7")' \
    && Rscript -e 'devtools::install_version("class", version="7.3-14")' \
    && Rscript -e 'devtools::install_version("mvtnorm", version="1.0.8")' \
    && Rscript -e 'devtools::install_version("multcomp", version="1.4-8")' \
    && Rscript -e 'devtools::install_version("coin", version="1.2.2")' \
    && Rscript -e 'devtools::install_version("party", version="1.0-25",upgrade=FALSE)' \
    && Rscript -e 'devtools::install_version("e1071", version="1.6-7")' \
    && Rscript -e 'devtools::install_version("randomForest", version="4.6-12")'



########################################
# Install FSL
# from neurodocker 

RUN export ND_ENTRYPOINT="/neurodocker/startup.sh" \
    && apt-get update -qq \
    && apt-get install -y -q --no-install-recommends \
           apt-utils \
           bzip2 \
           ca-certificates \
           curl \
           locales \
           unzip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen \
    && dpkg-reconfigure --frontend=noninteractive locales \
    && update-locale LANG="en_US.UTF-8" \
    && chmod 777 /opt && chmod a+s /opt \
    && mkdir -p /neurodocker \
    && if [ ! -f "$ND_ENTRYPOINT" ]; then \
         echo '#!/usr/bin/env bash' >> "$ND_ENTRYPOINT" \
    &&   echo 'set -e' >> "$ND_ENTRYPOINT" \
    &&   echo 'export USER="${USER:=`whoami`}"' >> "$ND_ENTRYPOINT" \
    &&   echo 'if [ -n "$1" ]; then "$@"; else /usr/bin/env bash; fi' >> "$ND_ENTRYPOINT"; \
    fi \
    && chmod -R 777 /neurodocker && chmod a+s /neurodocker

ENTRYPOINT ["/neurodocker/startup.sh"]

ENV FSLDIR="/opt/fsl-6.0.3" \
    PATH="/opt/fsl-6.0.3/bin:$PATH" \
    FSLOUTPUTTYPE="NIFTI_GZ" \
    FSLMULTIFILEQUIT="TRUE" \
    FSLTCLSH="/opt/fsl-6.0.3/bin/fsltclsh" \
    FSLWISH="/opt/fsl-6.0.3/bin/fslwish" \
    FSLLOCKDIR="" \
    FSLMACHINELIST="" \
    FSLREMOTECALL="" \
    FSLGECUDAQ="cuda.q"
RUN apt-get update -qq \
    && apt-get install -y -q --no-install-recommends \
           bc \
           dc \
           file \
           libfontconfig1 \
           libfreetype6 \
           libgl1-mesa-dev \
           libgl1-mesa-dri \
           libglu1-mesa-dev \
           libgomp1 \
           libice6 \
           libxcursor1 \
           libxft2 \
           libxinerama1 \
           libxrandr2 \
           libxrender1 \
           libxt6 \
           sudo \
           wget \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && echo "Downloading FSL ..." \
    && mkdir -p /opt/fsl-6.0.3 \
    && curl -fsSL --retry 5 https://fsl.fmrib.ox.ac.uk/fsldownloads/fsl-6.0.3-centos6_64.tar.gz \
    | tar -xz -C /opt/fsl-6.0.3 --strip-components 1 \
#    && sed -i '$iecho Some packages in this Docker container are non-free' $ND_ENTRYPOINT \
#    && sed -i '$iecho If you are considering commercial use of this container, please consult the relevant license:' $ND_ENTRYPOINT \
#    && sed -i '$iecho https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/Licence' $ND_ENTRYPOINT \
#    && sed -i '$isource $FSLDIR/etc/fslconf/fsl.sh' $ND_ENTRYPOINT \
    && echo "Installing FSL conda environment ..." \
    && bash /opt/fsl-6.0.3/etc/fslconf/fslpython_install.sh -f /opt/fsl-6.0.3


#############################################

# Install FIX

RUN wget http://www.fmrib.ox.ac.uk/~steve/ftp/fix.tar.gz \
    && tar -zxvf fix.tar.gz \ 
    && fixVersion=1.06.15 \
    && mv fix /opt/fix-${fixVersion} \
    && rm -f /opt/fix \
    && ln -s /opt/fix-${fixVersion} /opt/fix \
    && rm -rf fix.tar.gz

ENV PATH="/opt/fix:$PATH" \
    FSL_FIXDIR=/opt/fix

# edit fix setting.sh ...
RUN sed -i -e 's+FSL_FIX_R_CMD="${FSLDIR}/fslpython/envs/fslpython/bin/R"+FSL_FIX_R_CMD="/opt/fix/bin"+' /opt/fix/settings.sh \
    && sed -i -e 's+FSL_FIX_MCRROOT="/opt/fmrib/MATLAB/MATLAB_Compiler_Runtime"+FSL_FIX_MCRROOT="/opt/mcr"+' /opt/fix/settings.sh 


#############################################

# Install MCR

# From: usuresearch/matlab-runtime
# Download and install Matlab Compiler Runtime v93 (2017b)
# Install the MCR dependencies and some things we'll need and download the MCR
# from Mathworks -silently install it
# See http://www.mathworks.com/products/compiler/mcr/ for more info.
# Adapted from https://github.com/flywheel-apps/matlab-mcr

RUN apt-get -q update && \
    apt-get install -q -y --no-install-recommends \      
      xorg \
      unzip \
      wget \
      curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
    
# Install the MCR dependencies and some things we'll need and download the MCR
# from Mathworks -silently install it
RUN mkdir /mcr-install && \
    mkdir /opt/mcr && \
    cd /mcr-install && \
    wget -q http://ssd.mathworks.com/supportfiles/downloads/R2017b/deployment_files/R2017b/installers/glnxa64/MCR_R2017b_glnxa64_installer.zip && \
    unzip -q MCR_R2017b_glnxa64_installer.zip && \
    rm -f MCR_R2017b_glnxa64_installer.zip && \
    ./install -destinationFolder /opt/mcr -agreeToLicense yes -mode silent && \
    cd / && \
    rm -rf mcr-install

# Configure environment variables for MCR 
# ENV LD_LIBRARY_PATH /opt/mcr/v93/runtime/glnxa64:/opt/mcr/v93/bin/glnxa64:/opt/mcr/v93/sys/os/glnxa64. #skip this
ENV XAPPLRESDIR /opt/mcr/v93/X11/app-defaults

# Extra dependency for MCR
RUN add-apt-repository ppa:openjdk-r/ppa && \
    apt-get update && \
    apt-get install -y openjdk-8-jdk

#############################################

# Install HCPPipelines scripts from GitHub

RUN rm -vfR /var/lib/apt/lists/* \
    && apt-get update \
    && apt-get install -y -q --no-install-recommends git \
    && mkdir -p /scripts \
    && cd /scripts \
    && git clone https://github.com/Washington-University/HCPpipelines.git

ENV HCPPIPEDIR="/scripts/HCPpipelines/"

#############################################
# Patch issue with tr command
RUN ln -s /usr/bin/tr /bin/tr 

# interactive command...
# /scripts/HCPpipelines/ICAFIX/hcp_fix /inputs/MNINonLinear/Results/task-rest3_bold/task-rest3_bold.nii.gz 2000 FALSE
