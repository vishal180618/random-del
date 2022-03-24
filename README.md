
# Locust

  

### About the tool:

  

-   Write test scenarios in plain Python
-   Distributed and scalable - supports hundreds of thousands of concurrent users
-   Web-based UI
    
All these points make Locust infinitely expandable and very developer friendly.

  
### To deploy the load testing tasks, you do the following:

1.  Deploy a load testing master.
2.  Deploy a group of load testing workers. With these load testing workers, you can create a substantial amount of traffic for testing purposes.

#### About the load testing master

The Locust master is the entry point for executing the load testing tasks. The Locust master configuration specifies several elements, including the ports to be exposed by the container:
8089 for the web interface
5557 and 5558 for communicating with workers
You use a service to allow the Locust workers to easily discover and reliably communicate with the master, even if the master fails and is replaced with a new pod by the deployment. The service also includes a directive to create an external forwarding rule at the cluster level so that external traffic can access the cluster resources.
After you deploy the Locust master, you can open the web interface using the public IP address of the external forwarding rule. After you deploy the Locust workers, you can start the simulation and look at aggregate statistics through the Locust web interface.
#### About the load testing workers

The Locust workers execute the load testing tasks. You use a single deployment to create multiple pods. The pods are spread out across the Kubernetes cluster. Each pod uses environment variables to control configuration information, such as the hostname of the system under test and the hostname of the Locust master.

### Steps to deploy this to kubernetes cluster

Initializing common variables

    REGION=us-central1
	ZONE=${REGION}-b
	PROJECT=$(gcloud config get-value project)
	CLUSTER=gke-load-test
	TARGET=https://api.dev.ingress-egress.nexusplatform.co.uk/groupnexus/ingress/1.0.0
	SCOPE="https://www.googleapis.com/auth/cloud-platform"

  
  
Set the default zone and project ID so you don't have to specify these values in every subsequent command:

	gcloud config set compute/zone ${ZONE}
	gcloud config set project ${PROJECT}

 ### Creating the GKE cluster

  

 1. Create the GKE cluster:

		gcloud container clusters create $CLUSTER \
		   --zone $ZONE \
		   --scopes $SCOPE \
		   --enable-autoscaling --min-nodes "3" --max-nodes "10" \
		   --scopes=logging-write,storage-ro \
		   --addons HorizontalPodAutoscaling,HttpLoadBalancing

  

 2. Connect to the GKE cluster:

		gcloud container clusters get-credentials $CLUSTER \
		   --zone $ZONE \
		   --project $PROJECT

### Building the Docker image

	gcloud builds submit --tag gcr.io/$PROJECT/locust-tasks:1.x .


### Deploying the Locust master and worker nodes

Replace the target host and project ID with the deployed endpoint and project ID in the locust-master-controller.yaml and locust-worker-controller.yaml files:

	sed -i -e "s/\[TARGET_HOST\]/$TARGET/g" kubernetes-config/locust-master-controller.yaml
	sed -i -e "s/\[TARGET_HOST\]/$TARGET/g" kubernetes-config/locust-worker-controller.yaml
	sed -i -e "s/\[PROJECT_ID\]/$PROJECT/g" kubernetes-config/locust-master-controller.yaml
	sed -i -e "s/\[PROJECT_ID\]/$PROJECT/g" kubernetes-config/locust-worker-controller.yaml

  
  

Deploy the Locust master and worker nodes:

	kubectl apply -f kubernetes-config/locust-master-controller.yaml
	kubectl apply -f kubernetes-config/locust-master-service.yaml
	kubectl apply -f kubernetes-config/locust-worker-controller.yaml

  
Verify the Locust deployments:

	kubectl get pods -o wide

  
Scale the pool of Locust worker pods to 20.

	kubectl scale deployment/locust-worker --replicas=20

  

Example run with 1200 req/sec(report is in saved html file)
