FROM public.ecr.aws/docker/library/python:3.8.20-bullseye@sha256:11da1db2f485e2560be9817f92932a29f68fefc332b0167e8c51ae308ee23337

ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV PYTHONUNBUFFERED=1 \
    OPENBLAS_NUM_THREADS=1 \
    OMP_NUM_THREADS=1 \
    PYTHONPATH=/testbed/python \
    LC_ALL=C.UTF-8 \
    LANG=C.UTF-8

RUN sed -i '/bullseye-security/d; s|http://deb.debian.org|https://deb.debian.org|g' /etc/apt/sources.list \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get -o Acquire::Retries=5 update \
    && apt-get -o Acquire::Retries=5 install -y --no-install-recommends --no-upgrade \
       build-essential git ca-certificates libgsl-dev=2.6+dfsg-2 \
    && rm -rf /var/lib/apt/lists/*

RUN python -m pip install --no-cache-dir \
      pip==24.3.1 setuptools==59.6.0 wheel==0.42.0 \
    && python -m pip install --no-cache-dir \
      numpy==1.19.5 h5py==2.10.0 attrs==20.3.0 jsonschema==3.2.0 \
      svgwrite==1.4.3 networkx==2.5.1 decorator==4.4.2 \
      pytest==7.4.4 packaging==23.2 pluggy==1.3.0 iniconfig==2.0.0 \
      exceptiongroup==1.2.2 tomli==2.0.1 six==1.17.0 pyrsistent==0.20.0 \
    && python -m pip install --no-cache-dir --no-deps --no-build-isolation \
      kastore==0.2.2 msprime==0.7.4

ARG REPO_URL=https://github.com/tskit-dev/tskit.git
ARG BASE_COMMIT=eb8b4aebe42f39f7e536a603e8cd25d9cf104e22
ARG INSTANCE_ID=clb-pyc-046

RUN git init /testbed \
    && cd /testbed \
    && git remote add origin "${REPO_URL}" \
    && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" \
    && git checkout --detach FETCH_HEAD \
    && git submodule update --init --depth=1 c/subprojects/full-projects/kastore \
    && test "$(git -C c/subprojects/full-projects/kastore rev-parse HEAD)" = "67c5a6af4b85ea263e5cab854d94b691949e88ae" \
    && git remote remove origin \
    && git -C c/subprojects/full-projects/kastore remote remove origin \
    && git config core.autocrlf false \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"

WORKDIR /testbed
# This historical checkout has no root .gitignore. Ignore only generated native
# build outputs and Python caches through the untracked local Git exclude file.
RUN printf '%s\n' 'python/build/' 'python/*.so' '**/__pycache__/' '.pytest_cache/' >> .git/info/exclude \
    && cd python \
    && python setup.py build_ext --inplace \
    && python -c 'import tskit, msprime, _tskit; print(tskit.__version__, msprime.__version__); msprime.simulate(2, random_seed=1)' \
    && test -z "$(git -C /testbed status --porcelain)"

CMD ["/bin/bash"]
