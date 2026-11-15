FROM public.ecr.aws/docker/library/python:3.6.15-buster@sha256:56ba32a1c5aa030914cc957f1c9fe58d9d462bdf89a1e5f2b4dbdda68085327e
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV PYTHONUNBUFFERED=1 LC_ALL=C.UTF-8 LANG=C.UTF-8 PYTHONPATH=/testbed
RUN python -m pip install --no-cache-dir pip==21.3.1 setuptools==59.6.0 wheel==0.37.1 numpy==1.14.5 pytest==3.6.3 hypothesis==3.66.32 coverage==4.5.1 attrs==19.3.0 py==1.5.4 pluggy==0.6.0 atomicwrites==1.1.5 more-itertools==4.2.0 six==1.11.0 sortedcontainers==2.1.0
ARG REPO_URL=https://github.com/astrofrog/fast-histogram.git
ARG BASE_COMMIT=76b6f57a27f31cdefc23444f91611a00c9b4332d
ARG INSTANCE_ID=clb-pyc-160
RUN git init /testbed && cd /testbed && git remote add origin "${REPO_URL}" && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" && git checkout --detach "${BASE_COMMIT}" && git remote remove origin && git config core.autocrlf false && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"
WORKDIR /testbed
RUN python setup.py build_ext --inplace --force && python -c "import fast_histogram as pkg,fast_histogram._histogram_core as native,numpy; print(pkg.__version__,numpy.__version__,native.__file__); assert pkg.__version__ == '0.5.dev0'; assert numpy.__version__ == '1.14.5'; assert native.__file__.startswith('/testbed/fast_histogram/_histogram_core.')" && git checkout HEAD -- . && printf '%s\n' '/build/' '**/__pycache__/' '*.pyc' '*.so' '/fast_histogram.egg-info/' >> .git/info/exclude && test -z "$(git status --porcelain)"
CMD ["/bin/bash"]
