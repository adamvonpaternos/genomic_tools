FROM ubuntu:20.04 as base

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
    unzip \
    cmake \
    gcc \
    g++ \
    automake \
    make \
    zlib1g-dev \
    libncurses5-dev \
    zlibc \
    libxml2

COPY apps/containers/bcl2fastq2/bcl2fastq2-v2-20-0-tar.zip ./
RUN unzip bcl2fastq2-v2-20-0-tar.zip && \
    tar -zxvf bcl2fastq2-v2.20.0.422-Source.tar.gz && \
    mkdir /bcl2fastq/bcl2fastq-build

RUN /bcl2fastq/src/configure --with-cmake=/usr/bin/cmake
RUN make && make install

FROM base as bcl2fastq

LABEL maintainer="Adam von Paternos" \
    name="bcl2fastq" \
    description="Illumina demultiplexing software." \
    version="2.20.0" \
    url="https://support.illumina.com/softwaredownload.html?assetId=82660c4c-f46c-4743-8566-2437755e4329&assetDetails=bcl2fastq2-v2-20-0-tar.zip"

COPY --from=builder /usr/local/bin/bcl2fastq /usr/local/bin/
COPY --from=builder /usr/local/share/xsl /usr/local/share/xsl
COPY --from=builder /usr/local/share/css /usr/local/share/css

FROM bcl2fastq as app

LABEL env="prod"
RUN useradd -m -s /bin/bash appuser
USER appuser

ENTRYPOINT ["/usr/bin/dumb-init", "--", "/usr/local/bin/bcl2fastq"]