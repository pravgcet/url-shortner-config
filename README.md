## URL shortner
#### This is the configuration repository for url shortner.

### Features
1. The shortened url can be of 8 - 12 characters long.
2. The long url can be of lenght <= 1000 characters.

### Architecture.
The services are divided according to CQRS pattern.
- Writer service designed to handle conversion of long URL to short URL.
  - Writer service has 2 replicas deployed. Load gets distributed in round robin fashion by k8's ClusterIp
- Reader service is designed to handle all the read queries.
  - Reader service has 3 replicas deployed. Load gets distributed in round robin fashion by k8's ClusterIp

### Technologies used
1. ETCD for keep track of ID's counter.
2. Postgres database.
3. Redis for caching.
4. Kong API gateway to route requests to reader and writer services.

Commands to start:

### 1. Install Etcd, Redis and Postgres
`helm install all-services ./umbrella-chart --namespace urlshortner --create-namespace`

### 2. Install spring boot url-writer(for post and put) and url-reader(for get) requests
`helm install url-writer ./url-shortner-writer -n urlshortner`

`helm install url-reader ./url-shortner-reader -n urlshortner`

### 3. Install Kong as api gateway and ratelimiter. (Exposed to be used with load balancers as well)
`helm install kong kong/kong -n urlshortner --create-namespace --set proxy.type=LoadBalancer --set env.database=off`

`kubectl apply -f kong-ingress.yaml`

Expose port if running in minikube: `kubectl port-forward svc/kong-kong-proxy 8080:80 -n urlshortner`

## Init
#### After starting all the services, call api PUT: localhost:8080/writer/etcd/set with payload 
``` json
{
    "key":"/url_shortener/next_id",
    "value":"11111111111"
}
```
to set the initial id in etcd. This will keep getting incremented as services requests ids for short url creation.


## API's exposed
1. SetKeyInEtcd
```json
{
			"name": "SetKeyInEtcd",
			"request": {
				"method": "PUT",
				"header": [],
				"body": {
					"mode": "raw",
					"raw": "{\n    \"key\":\"/url_shortener/next_id\",\n    \"value\":\"11111111111\"\n}",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": "localhost:8080/writer/etcd/set"
			},
			"response": []
		}
```
2. CreateShortUrl
```json
		{
			"name": "CreateShortUrl",
			"request": {
				"method": "POST",
				"header": [],
				"body": {
					"mode": "raw",
					"raw": "{\"longUrl\":\"https://www.google.com/search?q=random+facts&oq=random+facts&gs_lcrp=EgZjaHJvbWUyDAgAEEUYORixAxiABDIHCAEQABiABDIHCAIQABiABDIHCAMQABiABDIHCAQQABiABDIHCAUQABiABDIHCAYQABiABDIHCAcQABiABDIHCAgQABiABDIHCAkQABiABNIBCDI3ODRqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8\"}",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": "localhost:8080/writer/url"
			},
			"response": []
		}
```
3. AddCustomUrl
```json
		{
			"name": "AddCustomUrl",
			"request": {
				"method": "POST",
				"header": [],
				"body": {
					"mode": "raw",
					"raw": "{\"longUrl\":\"https://www.google.com/search?q=random+facts&oq=random+facts&gs_lcrp=EgZjaHJvbWUyDAgAEEUYORixAxiABDIHCAEQABiABDIHCAIQABiABDIHCAMQABiABDIHCAQQABiABDIHCAUQABiABDIHCAYQABiABDIHCAcQABiABDIHCAgQABiABDIHCAkQABiABNIBCDI3ODRqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8\"}",
					"options": {
						"raw": {
							"language": "json"
						}
					}
				},
				"url": "localhost:8080/url/custom"
			},
			"response": []
		},
```

4. GetbackLongUrl
```json
		{
			"name": "GetbackLongUrl",
			"request": {
				"method": "GET",
				"header": [],
				"url": "http://localhost:8080/reader/url/mh7ik1"
			},
			"response": []
		}
```
5. GetAllUrls
```json
		{
			"name": "GetAllUrls",
			"request": {
				"method": "GET",
				"header": [],
				"url": "http://localhost:8080/reader/url/all"
			},
			"response": []
		}
```
