# ###############################################################################
#
#                     Knative-Kafka Controller Makefile
#
# ###############################################################################

# Project Directories
BUILD_ROOT:=$(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
BUILD_DIR:=$(BUILD_ROOT)/build
BUILD_TESTS:=true

# Application Names (Align With Knative Eventing Version)
APP_NAME=kafka-channel-controller
APP_VERSION=latest

# Build / Packaging / Deployment Configuration
APP_PATH=eu.gcr.io/kyma-project/incubator
APP_TAG=$(APP_NAME):$(APP_VERSION)

#
# Clean Build Image
#

clean:
	@echo 'Cleaning $(APP_NAME) Build Image'
	rm -f $(BUILD_DIR)/$(APP_NAME)

.PHONY: clean


#
# Dep / Fmt / Vet / Lint / Test / Sonar
#

dep:
	@echo 'Ensuring Dependencies'
	cd $(BUILD_ROOT); dep ensure -v

format:
	@echo 'Formatting Go Source'
	go fmt ./...

vet:
	@echo 'Vetting Packages'
	go vet -v ./cmd/...
	go vet -v ./internal/...
	go vet -v ./pkg/...

lint:
	@echo 'Linting Packages'
	golint ./internal/...
	golint ./pkg/...

test:
	@echo 'Testing Packages'
	cd $(BUILD_ROOT); mkdir -p $(BUILD_DIR); go test -v ./pkg/... -coverprofile ${BUILD_DIR}/coverage.out
	cd $(BUILD_ROOT); go tool cover -func=${BUILD_DIR}/coverage.out

.PHONY: dep format vet lint test


#
# Code / YAML Generation
#

manifests:
	@echo 'Generating Controllers (CRDs, RBAC, etc.)'
	go run vendor/sigs.k8s.io/controller-tools/cmd/controller-gen/main.go all

clients:
	@echo 'Generating DeepCopy & Clients'
	vendor/k8s.io/code-generator/generate-groups.sh all \
		github.com/kyma-incubator/knative-kafka/components/controller/pkg/client \
		github.com/kyma-incubator/knative-kafka/components/controller/pkg/apis  \
		knativekafka:v1alpha1

.PHONY: manifests clients


#
# Build The Binaries
#

build-native: clean
	@echo 'Building $(APP_NAME) (Native)'
	mkdir -p $(BUILD_DIR); cd $(BUILD_ROOT)/cmd; go build -o $(BUILD_DIR)/$(APP_NAME)

build-linux: clean
	@echo 'Building $(APP_NAME) (Linux)'
	mkdir -p $(BUILD_DIR); cd $(BUILD_ROOT)/cmd; CGO_ENABLED=0 GOOS=linux go build -o $(BUILD_DIR)/$(APP_NAME)

.PHONY: build-native build-linux


#
# Package & Push Docker Container
#

docker-build:
	@echo 'Building $(APP_NAME) Docker Container'
	docker build --rm -t $(APP_TAG) --build-arg RUN_TESTS=${BUILD_TESTS} -f ./Dockerfile ../..
	docker tag $(APP_TAG) $(APP_PATH)/$(APP_TAG)

docker-push: docker-build
	@echo 'Pushing $(APP_NAME) Docker Container'
	docker push $(APP_PATH)/$(APP_TAG)

.PHONY: docker-build docker-push

#
# Support Prow
#

ci-master: docker-push

ci-pr: docker-build

.PHONY: ci-release ci-pr
