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
      "pip<24.1" "setuptools<60" "wheel<0.42" \
      Cython==0.29.36 numpy==1.17.5 pybind11==2.6.1 pytest==6.2.5

ARG REPO_URL=https://github.com/scipy/scipy.git
ARG BASE_COMMIT=e13278e712ac33c6110b5d48a8da0acfa7768b71
ARG INSTANCE_ID=clb-pyc-041

LABEL org.opencontainers.image.title="Python-C benchmark environment for scipy/scipy" \
      org.opencontainers.image.description="Clean SciPy Base checkout for the official splprep fixed-knots crash regression" \
      org.opencontainers.image.source="https://github.com/scipy/scipy" \
      benchmark.instance_id="${INSTANCE_ID}" \
      benchmark.base_commit="${BASE_COMMIT}"

RUN git init /testbed \
    && cd /testbed \
    && git remote add origin "${REPO_URL}" \
    && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" \
    && git checkout --detach FETCH_HEAD \
    && git remote remove origin \
    && git config core.autocrlf false \
    && git reflog expire --expire=now --all \
    && git gc --prune=now \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"

WORKDIR /testbed

RUN python3.8 setup.py build_ext -i \
    && python3.8 -c "import scipy, scipy.interpolate; print(scipy.__version__)" \
    && git reset --hard HEAD \
    && rm -f \
      scipy/integrate/_dop-f2pywrappers.f \
      scipy/integrate/lsoda-f2pywrappers.f \
      scipy/integrate/vode-f2pywrappers.f \
    && test -z "$(git status --porcelain)"

CMD ["/bin/bash"]
