CHART_REPO := http://chartmuseum:8080
NAME := aps
OS := $(shell uname)
RELEASE_VERSION := "1.9.1"

setup:
	helm repo add aps https://introproventures.github.io/aps-chart

build:  clean
	helm dependency build
	helm lint

template: build
	helm template .

install: clean build
	helm install --name $(NAME) .

dry: 
	helm install --dry-run --debug --name $(NAME)-test . > dry-run.yaml

status:
	kubectl get pods	

upgrade: clean build
	helm upgrade --install $(NAME) .

delete:
	helm delete --purge $(NAME)

clean: 
	rm -rf charts/*.tgz
	rm -rf ${NAME}*.tgz
	rm -rf requirements.lock

release: setup clean build
ifeq ($(OS),Darwin)
	sed -i "" -e "s/version:.*/version: $(RELEASE_VERSION)/" Chart.yaml
	sed -i "" -e "s/tag:.*/tag: $(RELEASE_VERSION)/" values.yaml

else ifeq ($(OS),Linux)
	sed -i -e "s/version:.*/version: $(RELEASE_VERSION)/" Chart.yaml
	sed -i -e "s/tag:.*/tag: $(RELEASE_VERSION)/" values.yaml
endif

	helm package .
	curl --fail -u $(CHARTMUSEUM_CREDS_USR):$(CHARTMUSEUM_CREDS_PSW) --data-binary "@$(NAME)-$(RELEASE_VERSION).tgz" $(CHART_REPO)/api/charts
	rm -rf ${NAME}*.tgz

