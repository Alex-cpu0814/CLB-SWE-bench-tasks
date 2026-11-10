FROM public.ecr.aws/ubuntu/ubuntu:18.04@sha256:975b418c2c6c72bdfb978ac631c3fae2f42b752f5ab29dec706a01f3f6acc14b

ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       build-essential ca-certificates gfortran git libatlas-base-dev \
       python2.7 python2.7-dev python-nose python-pip python-setuptools \
    && rm -rf /var/lib/apt/lists/*

ARG REPO_URL=https://github.com/numpy/numpy.git
ARG BASE_COMMIT=b65b21c397381351965bf3f590054d2bccdd03d2
ARG INSTANCE_ID=clb-pyc-031
ARG PYTHON_VERSION=2.7
ARG NUMPY_VERSION=source-1.3.0.dev

LABEL org.opencontainers.image.title="Python-C benchmark environment for numpy/numpy" \
      org.opencontainers.image.description="Clean NumPy Base checkout for official choose broadcasting regression tests" \
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

RUN sed -i '5i from distutils.errors import DistutilsExecError' numpy/distutils/ccompiler.py \
    && python2.7 setup.py build_ext -i \
    && python2.7 -c "import numpy; print(numpy.__version__)" \
    && git reset --hard "${BASE_COMMIT}" \
    && git status --porcelain | sed 's/^?? //' >> .git/info/exclude \
    && test -z "$(git status --porcelain)"

CMD ["/bin/bash"]
