# SyncLyrics (multi-artist fix) - fast build
#
# Starts from the author's published SyncLyrics image (already compiled) and only
# applies the small patches in ./patches on top, so a Home Assistant install/update
# takes ~1-2 minutes instead of 15-20 minutes of compiling Python packages.
#
# UPSTREAM_VERSION pins the upstream image. When upstream releases a new SyncLyrics
# version, bump it here (and `version:` in config.yaml) - the build fails loudly if a
# patch no longer applies, instead of silently shipping without the fix.
#
# Home Assistant's Supervisor passes BUILD_ARCH (aarch64 / amd64) as a build arg.
ARG BUILD_ARCH=aarch64
ARG UPSTREAM_VERSION=2.4.0
FROM ghcr.io/anshulj999/synclyrics-ha-${BUILD_ARCH}:${UPSTREAM_VERSION}

COPY patches /patches
RUN apk add --no-cache --virtual .patch-deps git \
    && cd /app \
    && for p in /patches/*.patch; do echo "Applying $p"; git apply --verbose "$p"; done \
    && apk del .patch-deps \
    && rm -rf /patches

# Our run.sh adds the Musixmatch login options on top of upstream's
COPY run.sh /run.sh
RUN tr -d '\r' < /run.sh > /run.sh.tmp && mv /run.sh.tmp /run.sh && chmod a+x /run.sh

CMD [ "/run.sh" ]
