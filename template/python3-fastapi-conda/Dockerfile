FROM ghcr.io/openfaas/of-watchdog:0.9.11 as watchdog 

#points to last lts
FROM ubuntu:latest

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

COPY  --from=docker.io/mambaorg/micromamba:alpine /bin/micromamba /bin/micromamba

RUN mkdir -p /home/app/function                                                                                             && \
    touch /home/app/function/__init__.py                                                                                    && \
    chown -R app:app /home/app
    
COPY --chown=app:app function/environment.yml /home/app/environment.yml
COPY --chown=app:app function /home/app/function
COPY --chown=app:app index.py  /home/app/index.py

# Install basic commands and mamba
RUN apt-get update -qy                                                                                                      && \
    apt-get install -y --no-install-recommends ca-certificates wget bash bzip2                                              && \
    micromamba shell init -s bash -p ~/micromamba                                                                           && \
    apt-get clean autoremove --yes                                                                                          && \
    rm -rf /var/lib/{apt,dpkg,cache,log}                                                                                    && \
    chmod +x /usr/bin/micromamba                                                                                            && \
    micromamba create -f /home/app/environment.yml                                                                          && \                                                              
    rm -fr /srv/conda/pkgs                                                                                                  && \
    rm -fr /tmp/*

#For opensearch-client
RUN wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb && \
    dpkg -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb

USER app
ENV LD_LIBRARY_PATH=/usr/local/ssl/lib:$LD_LIBRARY_PATH

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

WORKDIR /home/app/

CMD ["fwatchdog"]
