PROJECT_NAME=orderproc5
DOCKER_REPOSITORY=gandigit
DOCKER_REPOSITORY_REMOTE=gandigit
DOCKER_USER=gandigit
DOCKER_PASSWORD=abcde

ROOT_FOLDER=$(pwd)
ROOT_FOLDER_CATALOG=../../catalogservice
ROOT_FOLDER_ORDER=../../orderservice
ROOT_FOLDER_WEB=../../ordproc-web

PROXY_NODE=9.204.168.81
ENTRY_POINT_URL=http://$(PROXY_NODE):$(shell kubectl get svc istio-ingressgateway -n istio-system -o json | jq .spec.ports[0].nodePort)
ENTRY_POINT_URL_UI=http://$(PROXY_NODE):$(shell kubectl get svc istio-ingressgateway -n istio-system -o json | jq .spec.ports[0].nodePort)
URL_SERVICE_CATALOG=http://$(PROXY_NODE):$(shell kubectl get svc istio-ingressgateway -n istio-system -o json | jq .spec.ports[0].nodePort)
URL_SERVICE_ORDER=http://$(PROXY_NODE):$(shell kubectl get svc istio-ingressgateway -n istio-system -o json | jq .spec.ports[0].nodePort)

DOCKER_IMAGE_CATALOG=$(DOCKER_REPOSITORY)/$(PROJECT_NAME)/catalog
DOCKER_IMAGE_ORDER=$(DOCKER_REPOSITORY)/$(PROJECT_NAME)/order
DOCKER_IMAGE_WEB=$(DOCKER_REPOSITORY)/$(PROJECT_NAME)/web

FORTIO_POD=$(shell kubectl get pod -n $(PROJECT_NAME) -l app=fortio -o jsonpath='{.items[0].metadata.name}')
DEBUG_POD=$(shell kubectl get pod -n $(PROJECT_NAME) -l app=curl -o jsonpath='{.items[0].metadata.name}')

init:
	-bash -c 'kubectl create ns $(PROJECT_NAME)'
	-bash -c 'kubectl label namespace $(PROJECT_NAME) istio-injection=enabled'
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./imagepolicy.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./debug.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./psp.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./ingress.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./config/fortio-deploy.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./destinationrules.yaml | kubectl apply -f -

clean:
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./destinationrules.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./messaging/redis.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./debug.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./config/fortio-deploy.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./ingress.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./psp.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' ./imagepolicy.yaml | kubectl delete -f -
	-bash -c 'kubectl delete ns $(PROJECT_NAME)'


init-app-catalog:
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_CATALOG/config/service.yaml | kubectl apply -f -

build-app-catalog:
	-cd ../catalogservice
	-./gradlew build
	-cd $ROOT_FOLDER
	-docker build -t $(DOCKER_IMAGE_CATALOG):v1.0.0 $ROOT_FOLDER_CATALOG

push-app-catalog:
	-docker login -u $(DOCKER_USER) -p $(DOCKER_PASSWORD)
	-docker push $(DOCKER_IMAGE_CATALOG):v1.0.0
	
deploy-app-catalog: build-app-catalog push-app-catalog
	-sed -e 's~<CONTAINER>~$(DOCKER_IMAGE_CATALOG):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_CATALOG/config/deployment.yaml  | kubectl apply -f -

clean-app-catalog:
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_CATALOG/config/service.yaml | kubectl delete -f -
	-sed -e 's~<CONTAINER>~$(DOCKER_IMAGE_CATALOG):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_CATALOG/config/deployment.yaml | kubectl delete -f -

	-docker stop $(DOCKER_IMAGE_CATALOG):v1.0.0
	-docker rm $(DOCKER_IMAGE_CATALOG):v1.0.0
	-docker rmi $(DOCKER_IMAGE_CATALOG):v1.0.0

test-app-catalog:
	-bash -c '$ROOT_FOLDER_CATALOG/config/test.sh $(ENTRY_POINT_URL)'


init-app-order:
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_ORDER/config/service.yaml | kubectl apply -f -

build-app-order:
	-cd ../orderservice
	-./gradlew build
	-cd $ROOT_FOLDER
	-docker build -t $(DOCKER_IMAGE_ORDER):v1.0.0 $ROOT_FOLDER_ORDER

push-app-order:
	-docker login -u $(DOCKER_USER) -p $(DOCKER_PASSWORD)
	-docker push $(DOCKER_IMAGE_ORDER):v1.0.0

deploy-app-order: build-app-order push-app-order
	-sed -e 's~<CONTAINER>~$(DOCKER_IMAGE_ORDER):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_ORDER/config/deployment.yaml  | kubectl apply -f -

clean-app-order:
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_ORDER/config/service.yaml | kubectl delete -f -
	-sed -e 's~<CONTAINER>~$(DOCKER_IMAGE_ORDER):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_ORDER/config/deployment.yaml | kubectl delete -f -

	-docker stop $(DOCKER_IMAGE_ORDER):v1.0.0
	-docker rm $(DOCKER_IMAGE_ORDER):v1.0.0
	-docker rmi $(DOCKER_IMAGE_ORDER):v1.0.0

test-app-order:
	-bash -c '$ROOT_FOLDER_ORDER/config/test.sh $(ENTRY_POINT_URL)'

init-app-web:
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_WEB/config/service.yaml | kubectl apply -f -
	-sed -e 's~<URL_SERVICE_CATALOG>~$(URL_SERVICE_CATALOG)~g' $ROOT_FOLDER_WEB/angularUI/src/environments/environment.ts.template > $ROOT_FOLDER_WEB/ui/src/environments/environment.ts.template
    -sed -e 's~<ORDER_SERVICE_URL>~$(URL_SERVICE_ORDER)~g' $ROOT_FOLDER_WEB/angularUI/src/environments/environment.ts.template > $ROOT_FOLDER_WEB/ui/src/environments/environment.ts
    -sed -e 's~<URL_SERVICE_CATALOG>~$(URL_SERVICE_CATALOG)~g' $ROOT_FOLDER_WEB/angularUI/src/environments/environment.prod.ts.template > $ROOT_FOLDER_WEB/ui/src/environments/environment.prod.ts.template
    -sed -e 's~<ORDER_SERVICE_URL>~$(URL_SERVICE_ORDER)~g' $ROOT_FOLDER_WEB/angularUI/src/environments/environment.prod.ts.template > $ROOT_FOLDER_WEB/ui/src/environments/environment.prod.ts

build-app-web:
	-cd $ROOT_FOLDER_WEB/angularUI
	-ng build
	-cp dist/ordproc-ui/* $ROOT_FOLDER_WEB/src/main/resources/static
	-cd $ROOT_FOLDER_WEB
	-./gradlew build

	-cd $ROOT_FOLDER
	-docker build -t $(DOCKER_IMAGE_WEB):v1.0.0 $ROOT_FOLDER_WEBs

push-app-web:
	-docker login -u $(DOCKER_USER) -p $(DOCKER_PASSWORD)
	-docker push $(DOCKER_IMAGE_WEB):v1.0.0

deploy-app-web: build-app-web push-app-web
	-sed -e 's~<CONTAINER>~$(DOCKER_IMAGE_WEB):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_WEB/config/deployment.yaml  | kubectl apply -f -

clean-app-web:
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_WEB/config/service.yaml | kubectl delete -f -
	-sed -e 's~<CONTAINER>~$(DOCKER_IMAGE_WEB):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_WEB/config/deployment.yaml | kubectl delete -f -

	-docker stop $(DOCKER_IMAGE_WEB):v1.0.0
	-docker rm $(DOCKER_IMAGE_WEB):v1.0.0
	-docker rmi $(DOCKER_IMAGE_WEB):v1.0.0

test-app-web:
	-bash -c '$ROOT_FOLDER_WEB/config/test.sh $(ENTRY_POINT_URL)'

init-all: init init-app-catalog init-app-order init-app-web
build-all: build-app-catalog build-app-order build-app-web
push-all: push-app-catalog push-app-order push-app-web
deploy-all: deploy-app-catalog deploy-app-order deploy-app-web
clean-all: clean clean-app-catalog clean-app-order clean-app-web
test-all: test-app-catalog test-app-order test-app-web

initnp:
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_CATALOG/config/networkpolicy.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_ORDER/config/networkpolicy.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_WEB/config/networkpolicy.yaml | kubectl apply -f -

cleannp:
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_CATALOG/config/networkpolicy.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_ORDER/config/networkpolicy.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT_NAME)~g' $ROOT_FOLDER_WEB/config/networkpolicy.yaml | kubectl delete -f -


WEB_POD_NAME=$(shell kubectl -n $(PROJECT_NAME) get pod -l app=$(PROJECT_NAME)-web-service -o jsonpath='{.items[0].metadata.name}')

WEB_access:
	$(shell kubectl -n $(PROJECT_NAME) port-forward $(WEB_POD_NAME) 9080:80)

restart-all:
	kubectl delete pods --all -n $(PROJECT_NAME)