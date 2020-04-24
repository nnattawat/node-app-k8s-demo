# Microservices deployment with k*83 demo

## services
![alt text](./diagram.png "Logo Title Text 1")

**Backend services**
- posts: for frontend to create a post
- comments: for frontend to create a comment
- query: for frontend to query data (it has a cached data from both posts and comments)
- event bus: sync data to query service when post or comment is created 
- moderation: internally to approve content

**Frontend**
- client: only talk to three service e.g. posts, comments, query

**Load Balancer**
- for LB, as we run it locally, we don't need create LoadBalancer service, which connects our cluster to cloud provider's LB 
- we use [ingress-nginx](https://github.com/kubernetes/ingress-nginx) for a ingress controller that match incoming request and redirecting to coresponding `ClusterIP` of a pod based on URL path. Similarly to `reverse-proxy`

### build services
```bash
docker build -t nnonsung/comments-nodejs .
docker push nnonsung/comments-nodejs

cd moderation
docker build -t nnonsung/moderation-nodejs .
docker push nnonsung/moderation-nodejs

cd posts
docker build -t nnonsung/posts-nodejs .
docker push nnonsung/posts-nodejs

cd query
docker build -t nnonsung/query-nodejs .
docker push nnonsung/query-nodejs

cd event-bus
docker build -t nnonsung/event-bus-nodejs .
docker push nnonsung/event-bus-nodejs

cd client
docker build -t nnonsung/client-react .
docker push nnonsung/client-react
```

### run k8s locally
```bash
# create microservice's deployments and clusterIP services
cd infra/k8s
kubectl apply -f comments-deployments.yaml
kubectl apply -f posts-deployments.yaml
kubectl apply -f query-deployments.yaml
kubectl apply -f moderation-deployments.yaml
kubectl apply -f event-bus-deployments.yaml
kubectl apply -f client-deployments.yaml

# Make sure you have installed ingress-nginx before applying ingress-service
kubectl apply -f ingress-service.yaml

# show all running resources
kubectl get all
```

Now you should be able to visit react app on `localhost`

#### testing posts service locally with NodePort
```bash
kubectl apply -f posts-service.yaml

# then you can hit POST http://localhost:[service-port]/posts
# for example:
curl --location --request POST 'http://localhost:30712/posts' \
--header 'Content-Type: application/json' \
--data-raw '{
	"title": "New sadf"
}'
```