# spilo-17:4.0-wiremind0 with a newer tzdata, and nothing else.
# PostgreSQL reads /usr/share/zoneinfo (the PGDG packages use --with-system-tzdata), so the tzdata
# package is the only change: PostgreSQL, Patroni, glibc and ICU stay identical (no REINDEX).
# A rebuild from postgres-appliance/ would also upgrade PostgreSQL, glibc and about 250 other packages.
ARG BASE_IMAGE=ghcr.io/wiremind/spilo-17:4.0-wiremind0@sha256:78051f58e1b08a9da5656901ab5d62562d20e0f77b9930360c0d9e2fe4258ab5
FROM ${BASE_IMAGE}

ARG BASE_IMAGE
# tzdata 2026e has the Manitoba rule. On 2026-10-08 it is only in jammy-proposed: set
# TZDATA_POCKET=jammy-updates when Ubuntu publishes it there.
ARG TZDATA_VERSION=2026e-0ubuntu0.22.04.1
ARG TZDATA_POCKET=jammy-proposed

# The build fails if another package changes, if a rule is missing, or if /etc/localtime moves.
RUN set -eux; \
    dpkg-query -W -f='${Package} ${Version}\n' > /tmp/packages-before; \
    if [ "${TZDATA_POCKET}" != "jammy-updates" ]; then \
        echo "deb http://archive.ubuntu.com/ubuntu/ ${TZDATA_POCKET} main" > /etc/apt/sources.list.d/tzdata.list; \
    fi; \
    apt-get update; \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends --only-upgrade "tzdata=${TZDATA_VERSION}"; \
    rm -f /etc/apt/sources.list.d/tzdata.list; \
    rm -rf /var/lib/apt/lists/*; \
    dpkg-query -W -f='${Package} ${Version}\n' > /tmp/packages-after; \
    test "$(diff /tmp/packages-before /tmp/packages-after | grep -c '^[<>]')" -eq 2; \
    grep -qx "tzdata ${TZDATA_VERSION}" /tmp/packages-after; \
    rm /tmp/packages-before /tmp/packages-after; \
    test "$(TZ=America/Vancouver date -d '2027-02-13 12:00 UTC' +%H:%M)" = "05:00"; \
    test "$(TZ=America/Edmonton date -d '2027-02-13 12:00 UTC' +%H:%M)" = "06:00"; \
    test "$(TZ=America/Winnipeg date -d '2027-02-13 12:00 UTC' +%H:%M)" = "07:00"; \
    test "$(readlink -f /etc/localtime)" = "/usr/share/zoneinfo/Etc/UTC"

LABEL org.opencontainers.image.base.name="${BASE_IMAGE}" \
      org.opencontainers.image.description="spilo-17:4.0-wiremind0 with tzdata ${TZDATA_VERSION}"
