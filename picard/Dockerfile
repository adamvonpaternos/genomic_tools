FROM openjdk:8 as base

ENV DEBIAN_FRONTEND=noninteractive \
    DEBCONF_NONINTERACTIVE_SEEN=true

RUN apt-get update && \
    apt-get install -y --no-install-suggests --no-install-recommends \
    build-essential \
    software-properties-common \
    dumb-init && \
    apt-get clean

FROM base as builder

RUN apt-get install -y --no-install-suggests --no-install-recommends \
    wget

RUN wget https://github.com/broadinstitute/picard/releases/download/2.23.3/picard.jar

FROM base as picard

LABEL maintainer="Adam von Paternos <adamvonpaternos@gmail.com>" \
    name="picard" \
    description="Picard is a set of command line tools for manipulating high-throughput sequencing (HTS) data and formats such as SAM/BAM/CRAM and VCF." \
    version="2.23.3" \
    url="https://github.com/broadinstitute/picard/releases/download/2.23.3/picard.jar"

COPY --from=builder /picard.jar /usr/local/bin/
ENV PICARD=/usr/local/bin/picard.jar

FROM picard as app

LABEL env="prod"
RUN useradd -m -s /bin/bash appuser
USER appuser

ENTRYPOINT ["/usr/bin/dumb-init", "--", "java", "-jar", "/usr/local/bin/picard.jar" ]
