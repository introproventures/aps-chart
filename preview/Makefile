OS := $(shell uname)
CURRENT := $(shell pwd)
NAME := $(shell basename $(CURRENT))

preview: 
ifeq ($(OS),Darwin)
	sed -i "" -e "s/version:.*/version: $(PREVIEW_VERSION)/" Chart.yaml
	sed -i "" -e "s/version:.*/version: $(PREVIEW_VERSION)/" ../$(APP_NAME)/Chart.yaml
else 
	sed -i -e "s/version:.*/version: $(PREVIEW_VERSION)/" Chart.yaml
	sed -i -e "s/version:.*/version: $(PREVIEW_VERSION)/" ../$(APP_NAME)/Chart.yaml
endif
	echo "  version: $(PREVIEW_VERSION)" >> requirements.yaml	
	${MAKE} build

clean:
	rm -rf charts/*.tgz
	rm -rf ${NAME}*.tgz

build: clean
	helm dependency build
	helm lint
