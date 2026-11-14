FROM public.ecr.aws/docker/library/python:3.8.20-bullseye@sha256:11da1db2f485e2560be9817f92932a29f68fefc332b0167e8c51ae308ee23337
ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV OPENBLAS_NUM_THREADS=1 OMP_NUM_THREADS=1 NPY_NUM_BUILD_JOBS=4 \
    PYTHONUNBUFFERED=1 LC_ALL=C.UTF-8 LANG=C.UTF-8 SCIPY_USE_PYTHRAN=0 \
    CFLAGS="-O0 -g0" CXXFLAGS="-O0 -g0" FFLAGS="-O0 -g0"
RUN sed -i '/bullseye-security/d; s|http://deb.debian.org|https://deb.debian.org|g' /etc/apt/sources.list \
    && apt-get -o Acquire::Retries=5 update \
    && apt-get -o Acquire::Retries=5 install -y --no-install-recommends --no-upgrade \
       gfortran libopenblas-dev pkg-config git ca-certificates \
    && rm -rf /var/lib/apt/lists/*
RUN python -m pip install --no-cache-dir pip==24.3.1 setuptools==59.6.0 wheel==0.42.0 \
    && python -m pip install --no-cache-dir numpy==1.19.5 Cython==0.29.36 \
       pybind11==2.6.1 pytest==7.4.4 packaging==23.2 \
       pluggy==1.3.0 iniconfig==2.0.0 exceptiongroup==1.2.2 tomli==2.0.1
ARG REPO_URL=https://github.com/scipy/scipy.git
ARG BASE_COMMIT=018986cb0260881827c6ef33cd2d804f54e39c85
ARG INSTANCE_ID=clb-pyc-048
RUN git init /testbed && cd /testbed \
    && git remote add origin "${REPO_URL}" \
    && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" \
    && git checkout --detach FETCH_HEAD \
    && git submodule update --init --depth=1 scipy/_lib/boost \
    && test "$(git -C scipy/_lib/boost rev-parse HEAD)" = "2110ce20e59917b85b12059b3ddd8b133549f662" \
    && git -C scipy/_lib/boost remote remove origin \
    && git remote remove origin \
    && git config core.autocrlf false \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"
WORKDIR /testbed
RUN python setup.py build_ext -i \
    && python -c 'import scipy, scipy.signal; print(scipy.__version__)' \
    && git restore --source=HEAD --worktree -- . \
    && rm -f scipy/integrate/_dop-f2pywrappers.f scipy/integrate/lsoda-f2pywrappers.f scipy/integrate/vode-f2pywrappers.f \
    && test -z "$(git status --porcelain)"
CMD ["/bin/bash"]
