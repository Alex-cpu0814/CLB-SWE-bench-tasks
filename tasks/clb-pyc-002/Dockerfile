FROM public.ecr.aws/docker/library/python:3.9-slim-bookworm@sha256:a02e9c5406c416c504d6c9a1a306ff4080c3173f1008d192f953bd20382a2d5c

ARG DEBIAN_FRONTEND=noninteractive
ARG NUMPY_VERSION=1.19.5

LABEL org.opencontainers.image.title="Python-C benchmark environment for scipy/scipy" \
      org.opencontainers.image.description="Clean Base checkout with cached native build; patches and grading assets are injected at evaluation time" \
      org.opencontainers.image.source="https://github.com/scipy/scipy"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ENV OPENBLAS_NUM_THREADS=1 \
    OMP_NUM_THREADS=1 \
    NPY_NUM_BUILD_JOBS=4 \
    PYTHONUNBUFFERED=1 \
    LC_ALL=C.UTF-8 \
    LANG=C.UTF-8

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        gfortran \
        git \
        liblapack-dev \
        libopenblas-dev \
        pkg-config \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN python -m pip install --no-cache-dir \
        "pip<24.1" \
        "setuptools<70" \
        "wheel<0.42" \
    && python -m pip install --no-cache-dir \
        "numpy==${NUMPY_VERSION}" \
        Cython==0.29.36 \
        pybind11==2.6.2 \
        pytest==6.2.5 \
        pytest-xdist==2.5.0

ARG EXTRA_APT_PACKAGES=
RUN if [[ -n "${EXTRA_APT_PACKAGES}" ]]; then \
        apt-get update \
        && apt-get install -y --no-install-recommends ${EXTRA_APT_PACKAGES} \
        && apt-get clean \
        && rm -rf /var/lib/apt/lists/*; \
    fi

ARG REPO_URL=https://github.com/scipy/scipy.git
ARG BASE_COMMIT=ff177d92b79167eb5de593c9de41d3ceb97ab189
ARG INSTANCE_ID=clb-pyc-002

LABEL benchmark.instance_id="${INSTANCE_ID}" \
      benchmark.base_commit="${BASE_COMMIT}"

RUN git init /testbed \
    && cd /testbed \
    && git remote add origin "${REPO_URL}" \
    && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" \
    && git checkout --detach FETCH_HEAD \
    && git remote remove origin \
    && git config core.autocrlf false \
    && git for-each-ref --format='delete %(refname)' refs/heads refs/remotes refs/tags \
        | git update-ref --stdin \
    && git reflog expire --expire=now --all \
    && git gc --prune=now \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}" \
    && test -z "$(git status --porcelain)" \
    && mkdir -p /benchmark /inputs /results

WORKDIR /testbed

# Cache the expensive native build. The repository ignores these outputs, so
# `git clean -fd` retains them while source changes trigger incremental rebuilds.
RUN python setup.py build_ext -i \
    && python -c "import scipy; print(scipy.__version__)" \
    && test -z "$(git status --porcelain)"

CMD ["/bin/bash"]
