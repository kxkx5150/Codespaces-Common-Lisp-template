FROM mcr.microsoft.com/vscode/devcontainers/base:ubuntu-22.04

ARG USERNAME=vscode
ARG NOVNC_VERSION=1.2.0
ARG WEBSOCKIFY_VERSION=0.9.0
ENV DEBIAN_FRONTEND noninteractive

ARG WIDHT=1024
ARG HEIGHT=768

ENV DISPLAY :0
ENV LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    LC_ALL=C.UTF-8 \
    DISPLAY_WIDTH=${WIDHT} \
    DISPLAY_HEIGHT=${HEIGHT}

# build variables
ARG SBCL_VERSION="2.3.0"
ARG SBCL_URL=https://prdownloads.sourceforge.net/sbcl/sbcl-${SBCL_VERSION}-source.tar.bz2
ARG QUICKLISP_VERSION="2022-04-01"
ARG QUICKLISP_URL=http://beta.quicklisp.org/dist/quicklisp/${QUICKLISP_VERSION}/distinfo.txt


RUN apt-get update && apt-get install -y --no-install-recommends build-essential \
    curl \
    wget \
    git \
    cmake \
    libblas-dev \
    libffi-dev \
    liblapack-dev \
    libz-dev \
    libzmq3-dev \
    rlwrap \
    sbcl \
    gfortran \
    ca-certificates \
    libzstd-dev \
    xauth uuid-dev \
    dbus dbus-x11 alsa-utils pulseaudio pulseaudio-utils mesa-utils x11-apps \
    python3 python-is-python3 python3-distutils python3-numpy python3-pip \
    xvfb x11vnc xdotool wget tar supervisor net-tools lxde xterm \
    && apt-get clean -y && rm -rf /var/lib/apt/lists


WORKDIR /src
RUN curl -LO ${SBCL_URL} && \
    tar -xf sbcl-${SBCL_VERSION}-source.tar.bz2 && \
    rm /src/sbcl-${SBCL_VERSION}-source.tar.bz2 && \
    cd /src/sbcl-${SBCL_VERSION} && \
    ln -s /src/sbcl-${SBCL_VERSION} /src/sbcl && \
    bash make.sh --fancy --with-sb-dynamic-core --with-sb-linkable-runtime && \
    (cd src/runtime && make libsbcl.a) && \
    bash install.sh && \
    cd -

# Install noVNC
RUN mkdir -p /usr/local/novnc \
    && curl -k -L -o /tmp/novnc.zip https://github.com/novnc/noVNC/archive/v${NOVNC_VERSION}.zip \
    && unzip /tmp/novnc.zip -d /usr/local/novnc/ \
    && cp /usr/local/novnc/noVNC-${NOVNC_VERSION}/vnc.html /usr/local/novnc/noVNC-${NOVNC_VERSION}/index.html \
    && curl -k -L -o /tmp/websockify.zip https://github.com/novnc/websockify/archive/v${WEBSOCKIFY_VERSION}.zip \
    && unzip /tmp/websockify.zip -d /usr/local/novnc/ \
    && ln -sf /usr/local/novnc/websockify-${WEBSOCKIFY_VERSION} /usr/local/novnc/noVNC-${NOVNC_VERSION}/utils/websockify \
    && rm -rf /tmp/novnc.zip /tmp/websockify.zip

RUN cd /usr/bin && sudo mv lxpolkit lxpolkit.bak

ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf

USER ${USERNAME}
RUN mkdir -p /home/${USERNAME}/workspace
WORKDIR /home/${USERNAME}/workspace


# Quicklisp setup
RUN wget -P /tmp/ 'https://beta.quicklisp.org/quicklisp.lisp' \
    && sbcl --noinform --non-interactive --load /tmp/quicklisp.lisp \
    --eval "(quicklisp-quickstart:install :dist-url \"${QUICKLISP_URL}\")" \
    && sbcl --noinform --non-interactive --load /home/${USERNAME}/quicklisp/setup.lisp \
    --eval '(ql-util:without-prompting (ql:add-to-init-file))' \
    && echo '#+quicklisp(push (truename "/usr/src") ql:*local-project-directories*)' >> /home/${USERNAME}/.sbclrc \
    && rm -f /tmp/quicklisp.lisp


USER root
COPY ./.inject_bashrc /root
RUN cat /root/.inject_bashrc >> /home/${USERNAME}/.bashrc

EXPOSE 8085
CMD ["/usr/bin/supervisord"]


