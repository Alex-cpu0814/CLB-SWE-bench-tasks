FROM public.ecr.aws/docker/library/python:3.8.20-bullseye@sha256:11da1db2f485e2560be9817f92932a29f68fefc332b0167e8c51ae308ee23337
ARG DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV PYTHONUNBUFFERED=1 LC_ALL=C.UTF-8 LANG=C.UTF-8 PYTHONPATH=/testbed/build/local SDL_VIDEODRIVER=x11 SDL_AUDIODRIVER=dummy PYGAME_HIDE_SUPPORT_PROMPT=1
RUN sed -i '/bullseye-security/d; s|http://deb.debian.org|https://deb.debian.org|g' /etc/apt/sources.list \
    && apt-get -o Acquire::Retries=5 update \
    && apt-get -o Acquire::Retries=5 install -y --no-install-recommends --no-upgrade --allow-downgrades libudev1=247.3-7+deb11u5 libudev-dev=247.3-7+deb11u5 gcc=4:10.2.1-1 g++=4:10.2.1-1 git ca-certificates xvfb=2:1.20.11-1+deb11u13 xauth=1:1.1-1 libsdl2-2.0-0=2.0.14+dfsg2-3+deb11u1 libsdl2-dev=2.0.14+dfsg2-3+deb11u1 libsdl2-image-dev=2.0.5+dfsg1-2 libsdl2-mixer-dev=2.0.4+dfsg1-3 libsdl2-ttf-dev=2.0.15+dfsg1-1 libfreetype6-dev=2.10.4+dfsg-1+deb11u1 libportmidi-dev=1:217-6 \
    && rm -rf /var/lib/apt/lists/*
RUN python -m pip install --no-cache-dir pip==24.3.1 setuptools==68.2.2 wheel==0.42.0 \
       pytest==7.4.4 packaging==23.2 pluggy==1.3.0 iniconfig==2.0.0 exceptiongroup==1.2.2 tomli==2.0.1
ARG REPO_URL=https://github.com/pygame/pygame.git
ARG BASE_COMMIT=71c90cdb76cd8b12a0969f5d6314315bd0223c28
RUN git init /testbed && cd /testbed \
    && git remote add origin "${REPO_URL}" \
    && git fetch --depth=1 --no-tags origin "${BASE_COMMIT}" \
    && git checkout --detach "${BASE_COMMIT}" \
    && git remote remove origin \
    && git config core.autocrlf false \
    && test "$(git rev-parse HEAD)" = "${BASE_COMMIT}"
WORKDIR /testbed
RUN python setup.py -auto -sdl2 -config \
    && python setup.py build --build-lib /testbed/build/local --build-temp /testbed/build/temp --force \
    && cp src_py/freesansbold.ttf /testbed/build/local/pygame/freesansbold.ttf \
    && python -c "import pygame,pygame._freetype; print(pygame.version.ver,pygame._freetype.__file__); assert pygame.version.ver == '1.9.5.dev0'; assert pygame._freetype.__file__.startswith('/testbed/build/local/pygame/')" \
    && git restore --source=HEAD --worktree -- . \
    && test -z "$(git status --porcelain)"
CMD ["/bin/bash"]
