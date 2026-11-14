FROM public.ecr.aws/docker/library/python:3.8.20-bullseye@sha256:11da1db2f485e2560be9817f92932a29f68fefc332b0167e8c51ae308ee23337
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV PYTHONUNBUFFERED=1 LC_ALL=C.UTF-8 LANG=C.UTF-8 OPENBLAS_NUM_THREADS=1
RUN python -m pip install --no-cache-dir pip==24.3.1 setuptools==59.6.0 wheel==0.42.0 \
    && python -m pip install --no-cache-dir numpy==1.19.5 pytest==7.4.4 \
       packaging==23.2 pluggy==1.3.0 iniconfig==2.0.0 exceptiongroup==1.2.2 tomli==2.0.1
ARG REPO_URL=https://github.com/CheetahTemplate3/cheetah3.git
ARG BASE_COMMIT=c86628cad04edc5167f9a9c8999abc62c6558e7b
ARG INSTANCE_ID=clb-pyc-049
RUN git init /testbed && cd /testbed && git remote add origin "${REPO_URL}" \
    && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" && git checkout --detach FETCH_HEAD \
    && git remote remove origin && git config core.autocrlf false \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"
WORKDIR /testbed
RUN python setup.py build_ext --inplace \
    && python -c 'import Cheetah._namemapper; print(Cheetah._namemapper.__file__)' \
    && test -z "$(git status --porcelain)"
CMD ["/bin/bash"]
