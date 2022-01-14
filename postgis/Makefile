NAME=timescaledb-postgis
# Default is to timescaledev to avoid unexpected push to the main repo
# Set ORG to timescale in the caller
ORG=timescaledev
PG_VER=pg12
POSTGIS_VERSION=2.5.5
PG_VER_NUMBER=$(shell echo $(PG_VER) | cut -c3-)
TS_VERSION=master
PREV_TS_VERSION=$(shell wget --quiet -O - https://raw.githubusercontent.com/timescale/timescaledb/${TS_VERSION}/version.config | grep update_from_version | sed -e 's!update_from_version = !!')
PREV_TS_IMAGE="timescale/timescaledb:$(PREV_TS_VERSION)-pg$(PG_VER_NUMBER)$(PREV_EXTRA)"
PREV_IMAGE=$(shell if docker pull $(PREV_TS_IMAGE) >/dev/null; then echo "$(PREV_TS_IMAGE)"; else echo "postgres:$(PG_VER_NUMBER)-alpine"; fi )

# Beta releases should not be tagged as latest, so BETA is used to track.
BETA=$(findstring rc,$(TS_VERSION))
PLATFORM=linux/386,linux/amd64,linux/arm/v6,linux/arm/v7,linux/arm64
# PUSH_MULTI can be set to nothing for dry-run without pushing during multi-arch build
PUSH_MUTLI=--push
TAG_VERSION=$(ORG)/$(NAME):$(TS_VERSION)-$(PG_VER)
TAG_LATEST=$(ORG)/$(NAME):latest-$(PG_VER)
TAG=-t $(TAG_VERSION) $(if $(BETA),,-t $(TAG_LATEST))

default: image

.multi_postgis_$(TS_VERSION)_$(PG_VER): Dockerfile
	docker buildx create --platform $(PLATFORM) --name multibuild --use
	docker buildx inspect multibuild --bootstrap
	docker buildx build --platform $(PLATFORM) --build-arg PG_VERSION_TAG=$(PG_VER) \
		--build-arg TS_VERSION=$(TS_VERSION) --build-arg POSTGIS_VERSION=$(POSTGIS_VERSION) $(TAG) $(PUSH_MUTLI) .
	touch .multi_$(TS_VERSION)_$(PG_VER)
	docker buildx rm multibuild

.build_postgis_$(TS_VERSION)_$(PG_VER): Dockerfile
	docker build --build-arg TS_VERSION=$(TS_VERSION) --build-arg POSTGIS_VERSION=$(POSTGIS_VERSION) --build-arg PG_VERSION_TAG=$(PG_VER) $(TAG) .
	touch .build_postgis_$(TS_VERSION)_$(PG_VER)

image: .build_postgis_$(TS_VERSION)_$(PG_VER)

push: image
	docker push $(TAG_VERSION)
	if [ -z "$(BETA)" ]; then \
		docker push $(TAG_LATEST); \
	fi

multi: .multi_postgis_$(TS_VERSION)_$(PG_VER)


clean:
	rm -f *~ .build_postgis_* .multi_postgis_*
	docker buildx rm multibuild

.PHONY: default image push multi clean
