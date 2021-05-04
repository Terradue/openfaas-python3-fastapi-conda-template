FROM openfaas/of-watchdog:0.7.2 as watchdog
FROM ubuntu:20.04

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog

RUN chmod +x /usr/bin/fwatchdog                                                     && \
    groupadd --gid 1002 app                                                         && \
    useradd --comment "Default user"                                                   \
    --create-home                                                                      \
    --gid 1002                                                                         \
    --no-log-init                                                                      \
    --shell /bin/bash                                                                  \
    --uid 1002 app                                                                  && \
    chown app /home/app

ENV PATH=$PATH:/home/app/.local/bin:/srv/conda/envs/env_openfaas/bin                    \
    MAMBA_ROOT_PREFIX=/srv/conda                                                        \
    fprocess="uvicorn index:app --host 0.0.0.0 --port 8000"                             \
    cgi_headers="true"                                                                  \
    mode="http"                                                                         \
    upstream_url="http://127.0.0.1:8000"

ADD function/environment.yml /tmp/environment.yml


# Install basic commands and mamba
RUN apt-get update                                                                                                          && \
    apt-get install -y ca-certificates wget bash bzip2                                                                      && \
    wget -qO- https://micromamba.snakepit.net/api/micromamba/linux-64/latest | tar -xvj bin/micromamba --strip-components=1 && \
    ./micromamba shell init -s bash -p ~/micromamba                                                                         && \
    apt-get clean autoremove --yes                                                                                          && \
    rm -rf /var/lib/{apt,dpkg,cache,log}                                                                                    && \
    cp ./micromamba /usr/bin                                                                                                && \
    micromamba create -f /tmp/environment.yml                                                                               && \                                                              
    rm -fr /srv/conda/pkgs                                                                                                  && \
    rm -fr /tmp/*


USER app

COPY --chown=app:app index.py  /home/app/index.py

RUN mkdir -p /home/app/function                                                                                             && \
    touch /home/app/function/__init__.py

COPY --chown=app:app function /home/app/function

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
