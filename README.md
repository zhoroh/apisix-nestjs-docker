# Manage NestJs Microservices APIs with Apache APISIX API Gateway


👉 To execute and customize the example project per your need shown in this post, here are the minimum requirements you need to install in your system:

- ➡️ [Node.js](https://nodejs.org/en/download)
- ➡️ [Curl](https://curl.se/download.html)
- ➡️ [Visual Studio 2022](https://visualstudio.microsoft.com/downloads/)
- ➡️ [Docker Desktop](https://docs.docker.com/desktop/windows/install/) - you need also [Docker desktop](https://www.docker.com/products/docker-desktop/) installed locally to complete this tutorial. It is available for [Windows](https://desktop.docker.com/win/edge/Docker%20Desktop%20Installer.exe) or [macOS](https://desktop.docker.com/mac/edge/Docker.dmg). 
Or install the [Docker ACI Integration CLI for Linux](https://docs.docker.com/engine/context/aci-integration/#install-the-docker-aci-integration-cli-on-linux).

# Clone the demo repository
``` bash
git clone https://github.com/zhoroh/apisix-nestjs-docker.git
```
Go to root directory of apisix-nestjs-docker
``` bash
cd apisix-nestjs-docker
```

## Build a multi-container APISIX via Docker CLI

You can start the application by running `docker compose` command from the root folder of the project:

``` bash
docker-compose up -d
```

Sample output:

``` bash
[+] Running 6/6
 ✔ Container apisix-nestjs-docker-backend-1           Running                                                                                                                                            0.0s 
 ✔ Container apisix-nestjs-docker-web1-1              Running                                                                                                                                            0.0s 
 ✔ Container apisix-nestjs-docker-etcd-1              Started                                                                                                                                            1.2s 
 ✔ Container apisix-nestjs-docker-apisix-dashboard-1  Running                                                                                                                                            0.0s 
 ✔ Container apisix-nestjs-docker-web2-1              Running                                                                                                                                            0.0s 
 ✔ Container apisix-nestjs-docker-apisix-1            Started                                                                                                                                            4.2s
```

Once the containers are running, navigate to `http://localhost:3003/greet` in your web browser and you will see the following output:

<img width="529" alt="Screenshot 2023-06-20 173346" src="https://github.com/zhoroh/apisix-nestjs-docker/assets/68103229/d462cfb5-8d29-451d-8790-9bf226f2188b">

# NestJs-Api
## Goal : manage access to my API via APISix So I can prevent intruders by
- Restricting who has access to the API
- Limit-count :  limits the number of requests to your service by a given count per time.

## Restricting who has access to the API
We need to create a Customer Configuration Object as shown below
``` bash

 curl http://127.0.0.1:9180/apisix/admin/consumers -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -i -d '
{
    "username": "oreoluwa",
    "plugins": {
        "basic-auth": {
            "username":"zhoroh",
            "password": "123455"
        }
    }
}'
```

``` bash
 curl http://127.0.0.1:9180/apisix/admin/consumers -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -i -d '
{
    "username": "bayo",
    "plugins": {
        "basic-auth": {
            "username":"bayo13",
            "password": "123456"
        }
    }
}'

```
``` bash
 curl http://127.0.0.1:9180/apisix/admin/consumers -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -i -d '
{
    "username": "elijah",
    "plugins": {
        "basic-auth": {
            "username":"elijah12",
            "password": "123454"
        }
    }
}'
```

Configuring the Plugin to the route

``` bash
curl http://127.0.0.1:9180/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/greet",
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "backend:3003": 1
        }
    },
    "plugins": {
        "basic-auth": {},
        "consumer-restriction": {
            "whitelist": [
                "oreoluwa",
                "bayo"
            ]
        }
    }
}'
```

Testing it out

case 1: 

``` bash
curl -u zhoroh:123455 http://127.0.0.1:9080/greet
```
Result:
``` bash
HTTP/1.1 200 OK
```

case 2: 

``` bash
curl -u bayo13:123456 http://127.0.0.1:9080/greet
```
Result:
``` bash
HTTP/1.1 200 OK
```

case 3: 

``` bash
curl -u elijah12:123454 http://127.0.0.1:9080/greet
```
Result:
``` bash
HTTP/1.1 403 Forbidden
...
{"message":"The consumer_name is forbidden."}
```

zhoroh and bayo13 had a status code of 200 because they have been whitelisted (they have access to call the route)

## Limit-Count
``` bash
 curl -i http://127.0.0.1:9180/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "methods": ["GET"],
    "uri": "/greet",
    "plugins": {
        "limit-count": {
            "count": 3,
            "time_window": 60,
            "rejected_code": 403,
            "rejected_msg": "Requests are too frequent, please try again later.",
            "key_type": "var",
            "key": "remote_addr"
        }
    },
    "upstream": {
      "type": "roundrobin",
      "nodes": {
       "backend:3003": 1
     }
   }
}'
```
Testing Out Within 60 seconds

First Time
``` bash
curl http://127.0.0.1:9080/greet -i
```
Result:
``` bash
HTTP/1.1 200 OK
```
Second Time
``` bash
curl http://127.0.0.1:9080/greet -i
```
Result:
``` bash
HTTP/1.1 200 OK
```
Third Time
``` bash
curl http://127.0.0.1:9080/greet -i
```
Result:
``` bash
HTTP/1.1 200 OK
```
Fourth Time
``` bash
curl http://127.0.0.1:9080/greet -i
```
Result:
``` bash
HTTP/1.1 403 Forbidden
...
{"error_msg":"Requests are too frequent, please try again later."}
```






