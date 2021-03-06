FROM nvidia/cuda:8.0-cudnn7-devel-ubuntu16.04

LABEL maintainer="autuanliu <autuanliu@163.com>"

USER root

# Define version
ARG ANACONDA3_VERSION=5.0.1

# Install some dependencies and tools
RUN apt-get update --fix-missing && \
    apt-get -yq dist-upgrade && \
    apt-get install -y --fix-missing \
    curl \
    grep \
    sed \
    tmux \
    nano \
    wget \ 
    bzip2 \
    ca-certificates \
    locales \
    fonts-liberation \
    libglib2.0-0 \
    libxext6 \
    libsm6 \
    libxrender1 \
    git \
    dpkg \
    mercurial \
    subversion && \
    apt-get clean && \
	apt-get autoremove && \
	rm -rf /var/lib/apt/lists/* 

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Install Tini
# https://github.com/krallin/tini
RUN wget -q https://github.com/krallin/tini/releases/download/v0.10.0/tini && \
    echo "1361527f39190a7338a0b434bd8c88ff7233ce7b9a4876f3315c22fce7eca1b0 *tini" | sha256sum -c - && \
    mv tini /usr/local/bin/tini && \
    chmod +x /usr/local/bin/tini

# Create autuanliu user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN useradd -m -s /bin/bash -N -u 1000 autuanliu

ENV PATH=/opt/conda/bin:$PATH \
    HOME=/home/autuanliu
    
# Install anaconda as autuanliu
# https://github.com/ContinuumIO/docker-images/blob/master/anaconda3/Dockerfile
RUN echo 'export PATH=/opt/conda/bin:$PATH' > /etc/profile.d/conda.sh && \
    wget -nv https://repo.continuum.io/archive/Anaconda3-${ANACONDA3_VERSION}-Linux-x86_64.sh -O ~/anaconda.sh && \
    /bin/bash ~/anaconda.sh -b -p /opt/conda && \
    rm ~/anaconda.sh

# Install TensorFlow
# https://anaconda.org/anaconda/tensorflow-gpu
RUN pip install https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow_gpu-1.4.0-cp36-cp36m-linux_x86_64.whl

# Install pytorch
# http://pytorch.org/
RUN conda install -q -y pytorch torchvision -c pytorch

# Install Keras
RUN pip install keras

# Expose Ports for TensorBoard (6006), Ipython (8888)
EXPOSE 6006 8888
WORKDIR $HOME

# Configure container startup
ENTRYPOINT ["tini", "--"]
CMD [ "/bin/bash" ]

# Add local files as late as possible to avoid cache busting
COPY jupyter_notebook_config.py /etc/jupyter/

# Jupyter has issues with being run directly: https://github.com/ipython/ipython/issues/7062
COPY run_jupyter.sh ${HOME}

# Switch back to autuanliu to avoid accidental container runs as root
USER autuanliu

# Define share folder
RUN mkdir $HOME/sharef
