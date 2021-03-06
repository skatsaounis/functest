FROM opnfv/functest-core

ARG BRANCH=master
ARG OPENSTACK_TAG=master
ARG VIMS_TEST_TAG=release-130
ARG QUAFF_TAG=59213d6d8ee29433552bb75f505cdc96b0b18909
ARG CLOUDIFY_VIMS_TAG=gambia
ARG HEAT_VIMS_TAG=release-130
ARG VROUTER_TAG=fraser
ARG VROUTER_BP_TAG=9b76d46a388d32d4985797620e67c2ed3315b3e4
ARG JUJU_TAG=tags/juju-2.3.9
ARG JUJU_WAIT_TAG=2.6.4
ARG ABOT_CHARM=opnfv-fraser
ARG GODEPS_TAG=404a7e748cd352bb0d7449dedc645546eebbfc6e

ENV GOPATH /src/epc-requirements/go
ENV GOBIN /src/epc-requirements/go/bin
ENV PATH $GOBIN:$PATH

COPY clearwater-heat-singlenet-deps.patch /tmp/clearwater-heat-singlenet-deps.patch
RUN apk --no-cache add --update \
        ruby ruby-bundler ruby-irb ruby-rdoc \
        procps libxslt libxml2 zlib libffi go musl-dev && \
    apk --no-cache add --virtual .build-deps --update \
        ruby-dev g++ make libxslt-dev libxml2-dev zlib-dev libffi-dev && \
    wget -q -O- https://opendev.org/openstack/requirements/raw/branch/$OPENSTACK_TAG/upper-constraints.txt > upper-constraints.txt && \
    sed -i -E /^tempest==+.*$/d upper-constraints.txt && \
    case $(uname -m) in aarch*|arm*) sed -i -E /^PyNaCl=/d upper-constraints.txt ;; esac && \
    wget -q -O- https://git.opnfv.org/functest/plain/upper-constraints.txt?h=$BRANCH > upper-constraints.opnfv.txt && \
    sed -i -E /#egg=functest/d upper-constraints.opnfv.txt && \
    git clone --depth 1 -b $VIMS_TEST_TAG https://github.com/Metaswitch/clearwater-live-test /src/vims-test && \
    sed -i s/unf_ext\ \(.*\)/unf_ext\ \(0.0.7.4\)/g /src/vims-test/Gemfile.lock && \
    git init /src/vims-test/quaff && \
    (cd /src/vims-test/quaff && \
        git fetch --tags https://github.com/Metaswitch/quaff $QUAFF_TAG && \
        git checkout FETCH_HEAD) && \
    git init /src/vims-test/build-infra && \
    (cd /src/vims-test/build-infra && \
        git fetch --tags https://github.com/Metaswitch/clearwater-build-infra $VIMS_TEST_TAG && \
        git checkout FETCH_HEAD) && \
    git init /src/cloudify_vims && \
    (cd /src/cloudify_vims && \
        git fetch --tags https://github.com/Orange-OpenSource/opnfv-cloudify-clearwater.git $CLOUDIFY_VIMS_TAG && \
        git checkout FETCH_HEAD) && \
    git init  /src/heat_vims && \
    (cd /src/heat_vims && \
        git fetch --tags https://github.com/Metaswitch/clearwater-heat.git $HEAT_VIMS_TAG && \
        git checkout FETCH_HEAD && \
        patch -p1 < /tmp/clearwater-heat-singlenet-deps.patch) && \
    git init /src/opnfv-vnf-vyos-blueprint && \
    (cd /src/opnfv-vnf-vyos-blueprint && \
        git fetch --tags https://github.com/oolorg/opnfv-vnf-vyos-blueprint.git $VROUTER_BP_TAG && \
        git checkout FETCH_HEAD) && \
    mkdir -p /home/opnfv/functest/data/router && \
    git init /home/opnfv/functest/data/router/opnfv-vnf-data && \
    (cd /home/opnfv/functest/data/router/opnfv-vnf-data && \
        git fetch --tags https://github.com/oolorg/opnfv-vnf-data.git $VROUTER_TAG && \
        git checkout FETCH_HEAD) && \
    case "$(uname -m)" in \
    "armv7l" | "aarch64") ;; \
    *) \
        git init /src/epc-requirements/abot_charm && \
        (cd /src/epc-requirements/abot_charm && \
            git fetch --tags https://github.com/RebacaInc/abot_charm.git $ABOT_CHARM && \
            git checkout FETCH_HEAD) && \
        pip3 install --no-cache-dir --src /src -cupper-constraints.txt -cupper-constraints.opnfv.txt \
            juju-wait==$JUJU_WAIT_TAG && \
        go get -d github.com/rogpeppe/godeps && \
        (cd $GOPATH/src/github.com/rogpeppe/godeps && git checkout $GODEPS_TAG && go install -v github.com/rogpeppe/godeps) && \
        go get -d -v github.com/juju/juju/... || true && \
        (cd $GOPATH/src/github.com/juju/juju && git checkout $JUJU_TAG && godeps -u dependencies.tsv) && \
        go install -v github.com/juju/juju/... && \
        rm -r $GOPATH/src/ $GOPATH/pkg /src/epc-requirements/abot_charm/.git /root/.cache/go-build;; \
    esac && \
    (cd /src/vims-test && bundle config build.nokogiri --use-system-libraries && bundle install --system && bundle update rest-client) && \
    rm -r upper-constraints.txt upper-constraints.opnfv.txt /src/vims-test/.git /src/cloudify_vims/.git /src/heat_vims/.git /src/vims-test/quaff/.git \
        /src/vims-test/build-infra/.git /src/opnfv-vnf-vyos-blueprint/.git \
        /tmp/clearwater-heat-singlenet-deps.patch && \
    apk del .build-deps
COPY testcases.yaml /usr/lib/python3.8/site-packages/xtesting/ci/testcases.yaml
CMD ["run_tests", "-t", "all"]
