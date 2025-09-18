#:PHONY is a special directive in Makefiles. 
#It tells make that certain targets are not actual files, but just names of actions/commands to run.
.PHONY: run_website stop_website install_kind create_kind_cluster create_local_registry connect_registry_to_kind_network \
	connect_registry_to_kind create_kind_cluster_with_registry delete_kind_cluster delete_docker_registry, make_service, port_forward \
	create_ingress, run_service, run_application_in_kind_cluster 

run_website:
	docker build -t explorecalifornia.com . && \
		docker run --rm --name explorecalifornia.com -p 5000:80 -d explorecalifornia.com

stop_website:
	docker stop explorecalifornia.com

install_kind:
	curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.30.0/kind-linux-amd64 && \
	chmod +x ./kind && \
	sudo mv ./kind /usr/local/bin/kind

create_kind_cluster:
	kubectl config use-context kind-kind && \
	kind create cluster --name explorecalifornia.com --config ./kind_config.yaml || true && \
	kubectl get nodes

create_local_registry:
	if ! docker ps | grep -q 'local-registry'; \
	then docker run -d -p 5000:5000 --name local-registry --restart=always registry:2; \ 
	else echo "---> local-registry is already running. There's nothing to do here."; \
	fi

tag_docker_image:
	docker tag explorecalifornia.com localhost:5000/explorecalifornia.com

push_image_to_local_registry:
	docker push localhost:5000/explorecalifornia.com

connect_registry_to_kind_network:
	docker network connect kind local-registry || true

connect_registry_to_kind: connect_registry_to_kind_network
	kubectl apply -f ./kind_configmap.yaml

create_kind_cluster_with_registry:
	$(MAKE) create_local_registry && $(MAKE) create_kind_cluster && $(MAKE) connect_registry_to_kind

run_application_in_kind_cluster:
	kubectl apply -f deployment.yaml

delete_kind_cluster: delete_docker_registry
	kind delete cluster --name explorecalifornia.com

delete_docker_registry:
	docker stop local-registry && docker rm local-registry

make_service: # clusterip means the service is internal to the cluster and reachable only from pods inside the cluster.
	kubectl create service clusterip --dry-run=client --tcp=80:80 explorecalifornia.com --output=yaml > service.yaml
	


run_service:
	kubectl apply -f service.yaml

port_forward: #this does not needed if you have ingress
	kubectl port-forward service/explorecalifornia-svc 8888:80

create_ingress:
	kubectl create ingress explorecalifornia.com --rule="explorecalifornia.com/=explorecalifornia-svc:80" \
	 --dry-run=client --output=yaml > ingress.yaml && kubectl apply -f ingress.yaml

deploy_ingress_controller:
	kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

helm_show_chart:
	helm show all ./chart

   
