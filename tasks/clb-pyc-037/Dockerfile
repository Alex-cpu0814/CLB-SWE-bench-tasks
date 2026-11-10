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
    && for attempt in 1 2 3 4 5 6 7 8; do \
         apt-get -o Acquire::Retries=8 install -y --no-install-recommends \
           build-essential ca-certificates gfortran git libopenblas-dev \
           pkg-config python3.8-dev python3-pip python3-setuptools && break; \
         test "${attempt}" != 8; \
         apt-get -o Acquire::Retries=8 update; \
       done \
    && dpkg -s build-essential gfortran git libopenblas-dev python3.8-dev python3-pip >/dev/null \
    && rm -rf /var/lib/apt/lists/*

RUN python3.8 -m pip install --no-cache-dir \
      "pip<24.1" "setuptools<60" "wheel<0.42" \
      Cython==0.29.36 numpy==1.17.5 pytest==3.10.1

ARG REPO_URL=https://github.com/scipy/scipy.git
ARG BASE_COMMIT=bf05e8d3412e2d96ab6735b6935e2a65f2620c30
ARG INSTANCE_ID=clb-pyc-037

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

RUN python3.8 setup.py build_ext -i

RUN sed -i '/from numpy\._import_tools import PackageLoader/,/pkgload(verbose=SCIPY_IMPORT_VERBOSE,postpone=True)/d' \
      /testbed/build/py3k/scipy/__init__.py \
    && cd /testbed/build/py3k \
    && python3.8 -c "import scipy, scipy.ndimage; print(scipy.__version__)" \
    && test -z "$(git -C /testbed status --porcelain)"

CMD ["/bin/bash"]
