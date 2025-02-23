FROM python:3.11.11-slim-bullseye AS builder

WORKDIR /app

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update -qq \
  && apt-get -qqq install --no-install-recommends -y pkg-config gcc g++ \
  && apt-get upgrade --assume-yes \
  && apt-get clean \
  && rm -rf /var/lib/apt

# Install pip, Babel, and dependencies globally (removes venv issues)
RUN python -m pip install --upgrade pip setuptools wheel Babel==2.12.1 torch==2.0.1 --extra-index-url https://download.pytorch.org/whl/cpu "numpy<2"

COPY . .

# Compile translation files
RUN python scripts/compile_locales.py

# Install LibreTranslate from source
RUN pip install .

FROM python:3.11.11-slim-bullseye

ARG with_models=false
ARG models=""

RUN addgroup --system --gid 1032 libretranslate && adduser --system --uid 1032 libretranslate && mkdir -p /home/libretranslate/.local && chown -R libretranslate:libretranslate /home/libretranslate/.local
USER libretranslate

COPY --from=builder --chown=1032:1032 /app /app
WORKDIR /app

COPY --from=builder /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin

RUN if [ "$with_models" = "true" ]; then  \
  if [ ! -z "$models" ]; then \
  python scripts/install_models.py --load_only_lang_codes "$models";   \
  else \
  python scripts/install_models.py;  \
  fi \
  fi

EXPOSE 5000
ENTRYPOINT [ "libretranslate", "--host", "*" ]
