FROM public.ecr.aws/docker/library/python:3.6.15-buster@sha256:56ba32a1c5aa030914cc957f1c9fe58d9d462bdf89a1e5f2b4dbdda68085327e
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV PYTHONUNBUFFERED=1 LC_ALL=C.UTF-8 LANG=C.UTF-8 PYTHONPATH=/testbed
RUN python -m pip install --no-cache-dir pip==21.3.1 setuptools==59.6.0 wheel==0.37.1 setuptools_scm==3.3.3 numpy==1.15.4 pytest==5.4.2 hypothesis==5.16.0 attrs==19.3.0 sortedcontainers==2.1.0 py==1.8.1 pluggy==0.13.1 packaging==20.4 pyparsing==2.4.7 more-itertools==8.3.0 wcwidth==0.1.9 importlib-metadata==1.6.0 zipp==3.1.0 six==1.15.0
ARG REPO_URL=https://github.com/astrofrog/fast-histogram.git
ARG BASE_COMMIT=085efdad6570e25440beab781ef70e1f81605c12
ARG INSTANCE_ID=clb-pyc-159
RUN git init /testbed && cd /testbed && git remote add origin "${REPO_URL}" && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" && git checkout --detach "${BASE_COMMIT}" && git remote remove origin && git config core.autocrlf false && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"
ENV SETUPTOOLS_SCM_PRETEND_VERSION=0.8
WORKDIR /testbed
RUN python setup.py build_ext --inplace --force && python -c "import fast_histogram as pkg,fast_histogram._histogram_core as native,numpy; print(pkg.__version__,numpy.__version__,native.__file__); assert pkg.__version__ == '0.8'; assert numpy.__version__ == '1.15.4'; assert native.__file__.startswith('/testbed/fast_histogram/_histogram_core.')" && git checkout HEAD -- . && printf '%s\n' '/build/' '**/__pycache__/' '*.pyc' '*.so' '/fast_histogram/version.py' '/fast_histogram.egg-info/' >> .git/info/exclude && test -z "$(git status --porcelain)"
CMD ["/bin/bash"]
