FROM ubuntu:18.04 as base

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
    wget \
    libz-dev

RUN wget https://sourceforge.net/projects/bio-bwa/files/latest/download/bwa-0.7.17.tar.bz2 && \
    tar xvf bwa-0.7.17.tar.bz2 && \
    cd bwa-0.7.17 && \
    make && \
    cp bwa /usr/local/bin

FROM google/cloud-sdk:latest as references

LABEL maintainer="Adam von Paternos <adam.vonpaternos@gmail.com>" \
    name="references" \
    description="The Broad Institute's human genome reference file based on GRCh37." \
    version="b37" \
    url="gs://genomics-public-data/references/b37/Homo_sapiens_assembly19.fasta.gz" \
    env="prod"

RUN mkdir -p /usr/local/share/references/b37
RUN gsutil cp gs://genomics-public-data/references/b37/Homo_sapiens_assembly19.fasta.gz \
    /usr/local/share/references/b37/
RUN gunzip -k /usr/local/share/references/b37/Homo_sapiens_assembly19.fasta.gz

# index reference
COPY --from=builder /usr/local/bin/bwa /usr/local/bin
RUN bwa index -a bwtsw /usr/local/share/references/b37/Homo_sapiens_assembly19.fasta
RUN rm -rf /usr/local/bin/bwa

FROM base as bwa

LABEL maintainer="Adam von Paternos <adamvonpaternos@gmail.com>" \
    name="bwa" \
    description="BWA is a software package for mapping low-divergent sequences against a large reference genome, such as the human genome." \
    version="0.7.17" \
    url="https://sourceforge.net/projects/bio-bwa/files/latest/download/bwa-0.7.17.tar.bz2"

COPY --from=builder /usr/local/bin/bwa /usr/local/bin/
COPY --from=references /usr/local/share/references /usr/local/share/references

FROM bwa as app

LABEL env="prod"
RUN useradd -m -s /bin/bash appuser
USER appuser

ENTRYPOINT ["/usr/bin/dumb-init", "--", "bwa"]
