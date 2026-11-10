FROM public.ecr.aws/docker/library/ubuntu:20.04@sha256:8feb4d8ca5354def3d8fce243717141ce31e2c428701f6682bd2fafe15388214

ARG DEBIAN_FRONTEND=noninteractive
ARG EXTRA_APT_PACKAGES=

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV NPY_NUM_BUILD_JOBS=4 \
    OPENBLAS_NUM_THREADS=1 \
    OMP_NUM_THREADS=1 \
    PYTHONUNBUFFERED=1 \
    LC_ALL=C.UTF-8 \
    LANG=C.UTF-8

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       build-essential ca-certificates gfortran git libopenblas-dev \
       pkg-config python3.8-dbg python3.8-dev python3-pip python3-setuptools \
    && if [[ -n "${EXTRA_APT_PACKAGES}" ]]; then apt-get install -y --no-install-recommends ${EXTRA_APT_PACKAGES}; fi \
    && rm -rf /var/lib/apt/lists/*

RUN python3.8-dbg -m pip install --no-cache-dir \
      "pip<24.1" "setuptools<70" "wheel<0.42" \
      Cython==0.29.36 hypothesis==5.14.0 pytz==2020.1 \
      pytest==5.4.2 pytest-env==0.6.2 pytest-leaks==0.3.1

ARG REPO_URL=https://github.com/numpy/numpy.git
ARG BASE_COMMIT=5e8ad11b75245caf2d158eacf369b011ef262795
ARG INSTANCE_ID=clb-pyc-010
ARG PYTHON_VERSION=3.8-dbg
ARG NUMPY_VERSION=source-1.20.0.dev0

LABEL org.opencontainers.image.title="Python-C benchmark environment for numpy/numpy" \
      org.opencontainers.image.description="Clean NumPy Base checkout for official out-identity regression tests" \
      org.opencontainers.image.source="https://github.com/numpy/numpy" \
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
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}" \
    && mkdir -p /benchmark /inputs /results

WORKDIR /testbed

RUN python3.8-dbg setup.py build_ext -i \
    && python3.8-dbg -c "import numpy; print(numpy.__version__)" \
    && test -z "$(git status --porcelain)"

CMD ["/bin/bash"]
