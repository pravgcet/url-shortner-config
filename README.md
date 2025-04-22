## URL shortner
#### This is the configuration repository for url shortner.

### Features
1. The shortened url can be of 8 - 12 characters long.
2. The long url can be of lenght <= 1000 characters.

### Architecture.
The services are divided according to CQRS pattern.
- Writer service designed to handle conversion of long URL to short URL.
- Reader service is designed to handle all the read queries.

### Technologies used
1. ETCD for keep track of ID's counter.
2. Postgres database.
3. Redis for caching.
4. Kong API gateway to route requests to reader and writer services.
