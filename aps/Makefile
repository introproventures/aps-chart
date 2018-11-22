CHART_REPOSITORY := $(or $(CHART_REPOSITORY),http://jenkins-x-chartmuseum:8080)
CURRENT := $(shell pwd)
NAME := $(or $(APP_NAME), $(shell basename $(CURRENT)))
OS := $(shell uname)
RELEASE_VERSION := $(or $(shell cat VERSION), $(shell sed -n 's/^version: //p' Chart.yaml))

RELEASE_BRANCH := $(or $(RELEASE_BRANCH),master)
RELEASE_GREP_EXPR := '^[Rr]elease'

GITHUB_CHARTS_REPO := $(or $(GITHUB_CHARTS_REPO),$(shell git config --get remote.origin.url))
GITHUB_CHARTS_BRANCH := $(or $(GITHUB_CHARTS_BRANCH),gh-pages)

CHARTMUSEUM_GS_BUCKET := $(or $(CHARTMUSEUM_GS_BUCKET),$(ORG)-chartmuseum)

PROMOTE_HELM_REPO_URL := $(or $(PROMOTE_HELM_REPO_URL),$(CHART_REPOSITORY))
PROMOTE_APP := $(PROMOTE_APP)
PROMOTE_ALIAS := $(or $(PROMOTE_ALIAS),$(APP_NAME))
PROMOTE_ENV := $(PROMOTE_ENV)
PROMOTE_NAMESPACE := $(PROMOTE_NAMESPACE)
PROMOTE_ALL_AUTO := $(or $(PROMOTE_ALL_AUTO),true)
PROMOTE_NO_WAIT := $(or $(PROMOTE_NO_WAIT),false)
PROMOTE_NO_POLL := $(or $(PROMOTE_NO_POLL),false)
PROMOTE_NO_MERGE := $(or $(PROMOTE_NO_MERGE),false)
PROMOTE_VERBOSE := $(or $(PROMOTE_VERBOSE),false)

$(NAME)-$(RELEASE_VERSION).tgz: 
	${MAKE} package

VERSION: 
	$(shell jx-release-version > VERSION)

git-rev-list:
	$(eval REV = $(shell git rev-list --tags --max-count=1 --grep $(RELEASE_GREP_EXPR)))
	$(eval PREVIOUS_REV = $(shell git rev-list --tags --max-count=1 --skip=1 --grep $(RELEASE_GREP_EXPR)))
	$(eval REV_TAG = $(shell git describe ${PREVIOUS_REV}))
	$(eval PREVIOUS_REV_TAG = $(shell git describe ${REV}))
	@echo Found commits between $(PREVIOUS_REV_TAG) and $(REV_TAG) tags:
	git rev-list $(PREVIOUS_REV)..$(REV) --first-parent --pretty

init:
	helm init --client-only
	helm repo add chart-repo $(CHART_REPOSITORY)

clean:
	rm VERSION | true
	rm -rf charts/*.tgz
	rm -rf ${NAME}*.tgz

build: clean
	helm dependency build
	helm lint

template: build
	helm template .

next-version: VERSION
	sed -i -e "s/version:.*/version: $(shell cat VERSION)/" Chart.yaml
	@echo Using next release version $(shell cat VERSION)

tag: VERSION
	git status
	jx step tag --charts-dir .

credentials: 
	git config --global credential.helper store
	jx step git credentials

checkout: credentials
	@echo "ensure we're not on a detached head"
	git checkout $(RELEASE_BRANCH) 

package: build 
	helm package .

install: clean build
	helm install . --name ${NAME}

upgrade: clean build
	helm upgrade --install ${NAME} .

test: 
	helm upgrade --dry-run --debug --install $(NAME) .

status: 
	kubectl get pods

watch: 
	kubectl get pods -w

events:
	kubectl get events --sort-by=.metadata.creationTimestamp -w

delete:
	helm delete --purge ${NAME}

chartmuseum: $(NAME)-$(RELEASE_VERSION).tgz
	curl --fail -u $(CHARTMUSEUM_CREDS_USR):$(CHARTMUSEUM_CREDS_PSW) --data-binary "@$(NAME)-$(RELEASE_VERSION).tgz" $(CHART_REPOSITORY)/api/charts
	rm -rf ${NAME}*.tgz%
	
# run this command inside 'cloud-sdk' container in Jenkinsfile pipeline
gs-bucket: 
	curl --fail -L $(CHART_REPOSITORY)/index.yaml | gsutil cp - "gs://$(CHARTMUSEUM_GS_BUCKET)/index.yaml"
	
github: $(NAME)-$(RELEASE_VERSION).tgz
	$(eval GITHUB_CHARTS_DIR := $(shell basename $(GITHUB_CHARTS_REPO) .git))
	git clone -b "$(GITHUB_CHARTS_BRANCH)" "$(GITHUB_CHARTS_REPO)" $(GITHUB_CHARTS_DIR)
	cp "$(NAME)-$(RELEASE_VERSION).tgz" $(GITHUB_CHARTS_DIR)
	cd $(GITHUB_CHARTS_DIR) && \
	   helm repo index . && \
	   git add . && \
	   git status && \
	   git commit -m "release $(NAME)-$(RELEASE_VERSION).tgz" && \
	   git push origin "$(GITHUB_CHARTS_BRANCH)"
	rm -rf $(GITHUB_CHARTS_DIR)
	   
release:
	jx step helm release 

promote:
	@echo Doing promote to environments....
	jx promote -b \
		--all-auto=${PROMOTE_ALL_AUTO} \
		--timeout='1h' \
		--version='${RELEASE_VERSION}' \
		--alias='${PROMOTE_ALIAS}' \
		--app='${PROMOTE_APP}' \
		--env='${PROMOTE_ENV}' \
		--namespace='${PROMOTE_NAMESPACE}' \
		--no-merge=${PROMOTE_NO_MERGE} \
		--no-poll=${PROMOTE_NO_POLL} \
		--no-wait=${PROMOTE_NO_WAIT} \
		--helm-repo-url='${PROMOTE_HELM_REPO_URL}'

changelog: git-rev-list
	@echo Creating Github changelog for release: $(RELEASE_VERSION)
	jx step changelog \
		--version v$(RELEASE_VERSION) \
		--generate-yaml=true \
		--rev=$(REV) \
		--previous-rev=$(PREVIOUS_REV)
