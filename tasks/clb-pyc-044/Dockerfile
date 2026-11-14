FROM public.ecr.aws/ubuntu/ubuntu:20.04@sha256:7e9558e1203a9a2b0967554abf9ebdc5552f29013b01f6e7271bf2d776d51234

ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV OPENBLAS_NUM_THREADS=1 \
    OMP_NUM_THREADS=1 \
    NPY_NUM_BUILD_JOBS=4 \
    PYTHONUNBUFFERED=1 \
    LC_ALL=C.UTF-8 \
    LANG=C.UTF-8

RUN sed -i '/focal-backports/d; s|http://archive.ubuntu.com/ubuntu|http://security.ubuntu.com/ubuntu|g' /etc/apt/sources.list \
    && apt-get -o Acquire::Retries=8 update \
    && apt-get -o Acquire::Retries=8 install -y --no-install-recommends \
      build-essential ca-certificates gfortran git libopenblas-dev \
      pkg-config python3.8-dev python3-pip python3-setuptools \
    && rm -rf /var/lib/apt/lists/*

RUN python3.8 -m pip install --no-cache-dir \
      "pip<24.1" "setuptools<60" "wheel<0.42"

RUN apt-get -o Acquire::Retries=8 update \
    && apt-get -o Acquire::Retries=8 install -y --no-install-recommends \
      curl libssl-dev zlib1g-dev libffi-dev libsqlite3-dev libbz2-dev \
      liblzma-dev libreadline-dev \
    && rm -rf /var/lib/apt/lists/*

# Use a Python ABI supported by this pre-Python-3.7 Cython release.
# The source checksum is published on python.org/downloads/release/python-3615/.
RUN curl --fail --location --retry 5 \
      https://www.python.org/ftp/python/3.6.15/Python-3.6.15.tar.xz \
      --output /tmp/Python-3.6.15.tar.xz \
    && echo 'bc04aa6c2a1a172a35012abd668538cd  /tmp/Python-3.6.15.tar.xz' | md5sum --check \
    && tar -xJf /tmp/Python-3.6.15.tar.xz -C /tmp \
    && cd /tmp/Python-3.6.15 \
    && ./configure --prefix=/opt/python36 --with-ensurepip=install \
    && make -j4 \
    && make install \
    && /opt/python36/bin/python3.6 -c 'import ssl, ctypes, sqlite3; print(ssl.OPENSSL_VERSION)' \
    && rm -rf /tmp/Python-3.6.15 /tmp/Python-3.6.15.tar.xz

ENV PATH=/opt/python36/bin:${PATH}

RUN python3.6 -m pip install --no-cache-dir --upgrade \
      pip==21.3.1 setuptools==59.6.0 wheel==0.37.1 \
    && python3.6 -m pip install --no-cache-dir \
      Cython==0.21.2 numpy==1.17.5 pytest==3.10.1 nose==1.3.7

# Cython 0.21's compiled FlowControl nodes cannot be deep-copied on Python 3.6.
# Its supported pure-Python compiler preserves the historical source semantics.
RUN python3.6 -m pip install --no-cache-dir --force-reinstall --no-binary=Cython \
      --global-option=--no-cython-compile Cython==0.21.2 \
    && python3.6 -c 'import Cython.Compiler.FlowControl as f; assert f.__file__.endswith(".py"); print(f.__file__)'

ARG REPO_URL=https://github.com/scipy/scipy.git
ARG BASE_COMMIT=fc6b387bc0c51b5b252712584580b4f0975498fb
ARG INSTANCE_ID=clb-pyc-044

RUN git init /testbed \
    && cd /testbed \
    && git remote add origin "${REPO_URL}" \
    && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" \
    && git checkout --detach FETCH_HEAD \
    && git remote remove origin \
    && git config core.autocrlf false \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"

WORKDIR /testbed

RUN python3.6 setup.py build_ext -i \
    && python3.6 -c "import scipy, scipy.ndimage; print(scipy.__version__)" \
    && git reset --hard HEAD

# Login shells reset PATH, so expose the selected interpreter/compiler there too.
RUN ln -s /opt/python36/bin/python3.6 /usr/local/bin/python3.6 \
    && ln -s /opt/python36/bin/cython /usr/local/bin/cython \
    && ln -s /opt/python36/bin/python3.6 /usr/local/bin/python

CMD ["/bin/bash"]
