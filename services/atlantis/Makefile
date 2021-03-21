BASE_DIR=base
STAGING_OVERLAY=overlays/staging
PRODUCTION_OVERLAY=overlays/prod
RELEASE_NAME=prometheus
RENDER?=false

.PHONY: install-dependencies
install-dependencies:
	helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
	helm repo update

.PHONY: template
template:
	rm -rf ${BASE_DIR}/${RELEASE_NAME}
	helm template ${RELEASE_NAME} \
		./chart \
		--namespace=sre \
		--api-versions=2 \
		--output-dir=${BASE_DIR} \
		--values=./${OVERLAY}/values.yaml \
  	--validate \
		--version 1.0.0

.PHONY: deploy-staging
deploy-staging: OVERLAY=${STAGING_OVERLAY}
deploy-staging: template
ifeq ($(RENDER),true)
	cd ${STAGING_OVERLAY} && kustomize build .
else
	cd ${STAGING_OVERLAY} && kustomize build . | kubectl apply -f -
endif

.PHONY: delete-staging
delete-staging: OVERLAY=${STAGING_OVERLAY}
delete-staging: template
ifeq ($(RENDER),true)
	cd ${STAGING_OVERLAY} && kustomize build .
else
	cd ${STAGING_OVERLAY} && kustomize build . | kubectl delete -f -
endif

.PHONY: deploy-production
deploy-production: OVERLAY=${PRODUCTION_OVERLAY}
deploy-production: template
ifeq ($(RENDER),true)
	cd ${PRODUCTION_OVERLAY} && kustomize build .
else
	cd ${PRODUCTION_OVERLAY} && kustomize build . | kubectl apply -f -
endif

.PHONY: delete-production
delete-production: OVERLAY=${PRODUCTION_OVERLAY}
delete-production: template
ifeq ($(RENDER),true)
	cd ${PRODUCTION_OVERLAY} && kustomize build .
else
	cd ${PRODUCTION_OVERLAY} && kustomize build . | kubectl delete -f -
endif

.PHONY: reload
reload:
	kubectl -n sre run prom-reloader --rm -ti --image=centos --restart=Never -- curl -XPOST http://prometheus.sre.svc.cluster.local:9090/-/reload

