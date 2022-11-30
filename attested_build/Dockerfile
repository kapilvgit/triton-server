
#
# Multistage build.
#
ARG TRITON_VERSION=2.19.0
ARG TRITON_CONTAINER_VERSION=22.02

ARG BASE_IMAGE=ubuntu:20.04
ARG BUILD_IMAGE=kapilvaswani/tritonserver_build:latest

############################################################################
##  Build image
############################################################################
FROM ${BUILD_IMAGE} AS tritonserver_build

############################################################################
##  Production stage: Create container with just inference server executable
############################################################################
FROM ${BASE_IMAGE}

ARG TRITON_VERSION
ARG TRITON_CONTAINER_VERSION

ENV TRITON_SERVER_VERSION ${TRITON_VERSION}
ENV NVIDIA_TRITON_SERVER_VERSION ${TRITON_CONTAINER_VERSION}
LABEL com.nvidia.tritonserver.version="${TRITON_SERVER_VERSION}"

ENV PATH /opt/tritonserver/bin:${PATH}

ENV TF_ADJUST_HUE_FUSED         1
ENV TF_ADJUST_SATURATION_FUSED  1
ENV TF_ENABLE_WINOGRAD_NONFUSED 1
ENV TF_AUTOTUNE_THRESHOLD       2
ENV TRITON_SERVER_GPU_ENABLED    0

# Create a user that can be used to run triton as
# non-root. Make sure that this user to given ID 1000. All server
# artifacts copied below are assign to this user.
ENV TRITON_SERVER_USER=triton-server
RUN userdel tensorrt-server > /dev/null 2>&1 || true &&     if ! id -u $TRITON_SERVER_USER > /dev/null 2>&1 ; then         useradd $TRITON_SERVER_USER;     fi &&     [ `id -u $TRITON_SERVER_USER` -eq 1000 ] &&     [ `id -g $TRITON_SERVER_USER` -eq 1000 ]

# Ensure apt-get won't prompt for selecting options
ENV DEBIAN_FRONTEND=noninteractive

# Common dependencies. FIXME (can any of these be conditional? For
# example libcurl only needed for GCS?)
RUN apt-get update &&     apt-get install -y --no-install-recommends             software-properties-common             libb64-0d             libcurl4-openssl-dev             libre2-5             git             dirmngr             libnuma-dev             curl               &&     rm -rf /var/lib/apt/lists/*

WORKDIR /opt/tritonserver
RUN rm -fr /opt/tritonserver/*
COPY --chown=1000:1000 nvidia_entrypoint.sh .
ENTRYPOINT ["/opt/tritonserver/nvidia_entrypoint.sh"]

ENV NVIDIA_BUILD_ID <unknown>
LABEL com.nvidia.build.id=<unknown>
LABEL com.nvidia.build.ref=

WORKDIR /opt/tritonserver
COPY --chown=1000:1000 LICENSE .
COPY --chown=1000:1000 TRITON_VERSION .
COPY --chown=1000:1000 NVIDIA_Deep_Learning_Container_License.pdf .

COPY --chown=1000:1000 --from=tritonserver_build /tmp/tritonbuild/install/bin/tritonserver bin/
COPY --chown=1000:1000 --from=tritonserver_build /tmp/tritonbuild/install/lib/libtritonserver.so lib/
COPY --chown=1000:1000 --from=tritonserver_build /tmp/tritonbuild/install/include/triton/core include/triton/core

# Top-level include/core not copied so --chown does not set it correctly,
# so explicit set on all of include
RUN chown -R triton-server:triton-server include

COPY --chown=1000:1000 --from=tritonserver_build /tmp/tritonbuild/install/third-party-src third-party-src

COPY --chown=1000:1000 --from=tritonserver_build /tmp/tritonbuild/install/backends backends
