FROM public.ecr.aws/docker/library/python:3.6.15-buster@sha256:56ba32a1c5aa030914cc957f1c9fe58d9d462bdf89a1e5f2b4dbdda68085327e
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV PYTHONUNBUFFERED=1 LC_ALL=C.UTF-8 LANG=C.UTF-8 PYTHONPATH=/testbed
RUN python -m pip install --no-cache-dir pip==21.3.1 setuptools==59.6.0 wheel==0.37.1 pytest==3.0.7 py==1.4.34
ARG REPO_URL=https://github.com/pygobject/pycairo.git
ARG BASE_COMMIT=eeea4de31f9c8f4496f5695ee0258040b401f0b4
ARG INSTANCE_ID=clb-pyc-141
RUN git init /testbed && cd /testbed && git remote add origin "${REPO_URL}" && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" && git checkout --detach "${BASE_COMMIT}" && git remote remove origin && git config core.autocrlf false && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"
WORKDIR /testbed
RUN python setup.py build_ext --inplace --force && python -c "import cairo,cairo._cairo as native; print(cairo.version,cairo.cairo_version_string(),native.__file__); assert cairo.version == '1.11.1'; assert native.__file__.startswith('/testbed/cairo/_cairo.')" && git checkout HEAD -- . && printf '%s\n' '/build/' '**/__pycache__/' '*.pyc' '*.so' '/cairo/config.h' '/pycairo.egg-info/' >> .git/info/exclude && test -z "$(git status --porcelain)"
CMD ["/bin/bash"]
