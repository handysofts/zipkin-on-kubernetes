# zipkin-on-kubernetes

Here you can find ready to use zipkin yaml file which can be deployed via kubernetes and uses elastic-search cluster for saving the date. Please follow below steps in order to get successfully created zipkin cluster:

1. Install ElasticSearch System:
```
kubectl apply -f https://download.elastic.co/downloads/eck/1.2.1/all-in-one.yaml
```
This will create namespace `elastic-system` and prepare everything in order to install and configure ES automatically.


2. Install elastic-search:
```
kubectl apply -f elasticsearch-kubernetes.yaml
```
This will create two valume for storing the data, install ElasticSearch on it and configure service. After successfully installation you will be able to access your ES on port `30200` from outside of your cluster. (`https://<YOUR_HOST>:30200`)

Username by default is `elastic` and password can be obtained using below command:
```
PASSWORD=$(kubectl -n elastic-system get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}') && echo $PASSWORD
```

In order to uninstall quickstart you can execute:
```
kubectl delete elasticsearch quickstart -n elastic-system && kubectl delete pv elasticsearch-pv-1 && kubectl delete pv elasticsearch-pv-0
```

If you want to uninstall whole ES system then use below command:
```
kubectl delete -f https://download.elastic.co/downloads/eck/1.2.1/all-in-one.yaml
```

Logs can be viewed using below command:
```
kubectl -n elastic-system logs -f statefulset.apps/elastic-operator
```

If you want to push some data to ES in order test if everything works properly or not you can use below command:
```
curl -u "elastic:$PASSWORD" -H "Content-Type: application/json" -XPOST "http://<YOUR_HOST>:30200/indexname/typename/optionalUniqueId" -d "{ \"test-field\" : \"test-value\"}"
```


3. Install Zipkin:
```
kubectl apply -f zipkin-kubernetes.yaml
```
This will create namespace `zipkin`, two replicas of zipkin server and service which can be accessed from the outside on port `30411` (can be accessed from `http://<YOUR_HOST>:30411`)


4. This step is optional but recommend to view what we have inside ES. So, as this step we can install and configure kibana to view our data inside ES:
```
kubectl apply -f kibana-kubernetes.yaml
```
This will install single kibana instance and export service on port `30601` (can be accessed from `http://<YOUR_HOST>:30601`)

Username by default is `elastic` and password can be obtained using below command:
```
kubectl -n elastic-system get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo
```

If you want to uninstall the kibana just execute below command:
```
kubectl delete kibana quickstart -n elastic-system
```