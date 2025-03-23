# Deploying wordpress using a custom docker image and kubernetes

## Scaling options:
If the load varies a lot day by day it is possible to use auto scaling that 
can be applied to the Wordpress service.

Wordpress deployment can also be manually scaled using replica sets.  

MySQL can be scaled manually and is generally done vertically or horizontally using 
replica sets.

Performance can be improved by Wordpress cache plugins for static pages.

## In Reality
We would use some sort of stats and metrics (graphite and graphana as an example)
to access performance, latency as well as the number of requests per second.

This would also be version controlled. Version control can be implemented using
a version file that's readable or alternatively git tags or a HISTORY.MD file.

## Deployment pipeline

I was unable to do a pipeline for deployment due to the time constraints however if 
I was to deploy this, the pipeline would look like:
- Build the images using cloud build or something similar.
- Run tests using travis or a test script.
- Deploy built image to staging.
- (Optional until specified) Deploy to production

Deployment can be done using gitlab or Jenkins or alternatives. 

#Deploy the artifacts

### Prerequisites:
- Minikube installed
- Kubernetes installed
- Docker installed

## How to run the Docker image provided:

Start by building the docker image locally:

```
docker build -t wordpress .
```

Alternatively pull the image using:
```
docker pull merishlalla/merishka-wordpress:wp
```

Once this step has been completed, you can run the image using:
```
docker run -p 8080:80 -d wordpress
```

You can then enter http://127.0.0.1:8080/index.php in your browser to check that 
Wordpress set up comes up as expected.

## How to run the Kubernetes image provided:

Assign MySQL password:

```
kubectl create secret generic mysql-pass --from-literal=password=YOUR_PASSWORD
```

This can be checked using ```kubectl get secrets``` 

Deploy the MySQL deployment:

```
kubectl apply -f kubernetes_files/mysql_deployment.yaml
```

Deploy the Wordpress deployment:

```
kubectl apply -f kubernetes_files/wordpress_deployment.yaml
```

The deployment statuses can be found by using:

```bash
kubectl get pods,pvc,services
```

(It is known that minikube doesn't always handle pvc the best so if the deployment 
 doesn't come up as expected, try to delete the pod.) 

To access workability:

```
minikube service wordpress --url
```

The expected output of this is something like: ```http://192.168.99.100:31096```

Paste this URL into your browser,and you're all set. 