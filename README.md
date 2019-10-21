# Docker Swarm integrated with Traefik 2.0 

## Docker Swarm cluster
These stack files were presented during the DevOps Congress in Wrocław, October 19th, 2019. During this event I gave a talk entitiled: 

### Container orchestration with Docker Swarm and Traefik.

The deployment scenario is straightforward. I assume that you already have a working Docker Swarm cluster and you can easily execute `docker node ls` command to get list of nodes consiting the cluster. 

```ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
toxdjn5jle3cd15f9m1jdwapw     vmname9qwey76t      Ready               Active                                  19.03.3
wd0ljmqto337s9257k6fhv2zm     vmname53mf3tp2      Ready               Active              Reachable           19.03.3
rzygxa2u1q6xaiijjxext4uuh *   vmnameemyh88xp      Ready               Active              Leader              19.03.3
jo55zm923572hi068athu8aai     vmnameoxe4v89o      Ready               Active              Reachable           19.03.3
```

If so, you can deploy a stack file containing two services: Traefik and Visualizer. However, before deploying the stack you should add label to any of your node or nodes to indicate where Traefik should be deployed. Due to the fact, that in the latest version of Traefk (2.0) integration with Consul is removed I recommend to add the label to the only one node. You can have a look on my previous example when I integrated Traefik with 3 nodes Consul cluster where SSL certificates: 

You can add label by executing this command. 

```docker node update vmname53mf3tp2 --label-add traefik=true```

Than you can verify it by inspecting the node: ```docker node inspect vmname53mf3tp2 --pretty```

## Docker stack files
The stack file can be deployed in following way:

```docker stack deploy -c stack-traefik.yml proxy --with-registry-auth --prune```

You can see what services have been deployed by executing command:

```docker service ls```

you should recieve following output:

```ID                  NAME                MODE                REPLICAS            IMAGE                              PORTS
rl7hooijmy0h        proxy_traefik       replicated          1/1                 traefik:2.0.2                      *:80->80/tcp, *:443->443/tcp
vs5yduxcmlq9        proxy_visualizer    replicated          1/1                 dockersamples/visualizer:latest
```

Than you can deploy application stack containing these two simple services: Nginx and a simple backend written in NodeJs. 

```docker stack deploy -c stack-app.yml app --with-registry-auth --prune```

Listing of the running services should looks as it follows:


```ID                  NAME                MODE                REPLICAS            IMAGE                              PORTS
xcmz8522e5pk        app_backend         replicated          2/2                 jakubhajek/nodejs-backend:latest
wiaag9k5azd6        app_frontend        replicated          2/2                 nginx:1.17-alpine
rl7hooijmy0h        proxy_traefik       replicated          1/1                 traefik:2.0.2                      *:80->80/tcp, *:443->443/tcp
vs5yduxcmlq9        proxy_visualizer    replicated          1/1                 dockersamples/visualizer:latest
```

## Used URL's in stack files
In my example I use following urls which can be probably updated in your use case: 

- `traefik.labs.cometari.eu`, 
- `viz.labs.cometari.eu`,  
- `node-app.labs.cometari.eu`

The login credentails to the protected sites are: `admin/admin`. 

## Slapper - a simple load testing tool
I use also [Slapper](https://github.com/ikruglov/slapper) to generate network traffic and test zero downtime depoyment which is more complex aspects. The tool use the endpoint url from file node-app.target. Here is the command I use to challenge my stack and generate some network traffic:

```docker run -it -v${PWD}:/app jakubhajek/slapper:1 slapper -targets /app/node-app.target -minY 30ms -maxY 200ms -timeout 30s -rate 50```

You can try to scale up e.g. frontend application by: 

```docker service scale app_frontend=8```

and then try to scale it down by simulating any issue with a container: 

```docker servie scale app_frontend=7```

Have fun with Traefik and container orchestration tools! 
# Docker Swarm integrated with Traefik 2.0 

## Docker Swarm cluster
These stack files were presented during the DevOps Congress in Wrocław, October 19th, 2019. During this event I gave a talk entitiled: 

### Container orchestration with Docker Swarm and Traefik.

The deployment scenario is straightforward. I assume that you already have a working Docker Swarm cluster and you can easily execute `docker node ls` command to get list of nodes consiting the cluster. 

```ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
toxdjn5jle3cd15f9m1jdwapw     vmname9qwey76t      Ready               Active                                  19.03.3
wd0ljmqto337s9257k6fhv2zm     vmname53mf3tp2      Ready               Active              Reachable           19.03.3
rzygxa2u1q6xaiijjxext4uuh *   vmnameemyh88xp      Ready               Active              Leader              19.03.3
jo55zm923572hi068athu8aai     vmnameoxe4v89o      Ready               Active              Reachable           19.03.3
```

If so, you can deploy a stack file containing two services: Traefik and Visualizer. However, before deploying the stack you should add label to any of your node or nodes to indicate where Traefik should be deployed. Due to the fact, that in the latest version of Traefk (2.0) integration with Consul is removed I recommend to add the label to the only one node. You can have a look on my previous example when I integrated Traefik with 3 nodes Consul cluster where SSL certificates: 

You can add label by executing this command. 

```docker node update vmname53mf3tp2 --label-add traefik=true```

Than you can verify it by inspecting the node: ```docker node inspect vmname53mf3tp2 --pretty```

## Docker stack files
The stack file can be deployed in following way:

```docker stack deploy -c stack-traefik.yml proxy --with-registry-auth --prune```

You can see what services have been deployed by executing command:

```docker service ls```

you should recieve following output:

```ID                  NAME                MODE                REPLICAS            IMAGE                              PORTS
rl7hooijmy0h        proxy_traefik       replicated          1/1                 traefik:2.0.2                      *:80->80/tcp, *:443->443/tcp
vs5yduxcmlq9        proxy_visualizer    replicated          1/1                 dockersamples/visualizer:latest
```

Than you can deploy application stack containing these two simple services: Nginx and a simple backend written in NodeJs. 

```docker stack deploy -c stack-app.yml app --with-registry-auth --prune```

Listing of the running services should looks as it follows:


```ID                  NAME                MODE                REPLICAS            IMAGE                              PORTS
xcmz8522e5pk        app_backend         replicated          2/2                 jakubhajek/nodejs-backend:latest
wiaag9k5azd6        app_frontend        replicated          2/2                 nginx:1.17-alpine
rl7hooijmy0h        proxy_traefik       replicated          1/1                 traefik:2.0.2                      *:80->80/tcp, *:443->443/tcp
vs5yduxcmlq9        proxy_visualizer    replicated          1/1                 dockersamples/visualizer:latest
```

## Used URL's in stack files
In my example I use following urls which can be probably updated in your use case: 

- `traefik.labs.cometari.eu`, 
- `viz.labs.cometari.eu`,  
- `node-app.labs.cometari.eu`

The login credentails to the protected sites are: `admin/admin`. 

## Slapper - a simple load testing tool
I use also [Slapper](https://github.com/ikruglov/slapper) to generate network traffic and test zero downtime depoyment which is more complex aspects. The tool use the endpoint url from file node-app.target. Here is the command I use to challenge my stack and generate some network traffic:

```docker run -it -v${PWD}:/app jakubhajek/slapper:1 slapper -targets /app/node-app.target -minY 30ms -maxY 200ms -timeout 30s -rate 50```

You can try to scale up e.g. frontend application by: 

```docker service scale app_frontend=8```

and then try to scale it down by simulating any issue with a container: 

```docker servie scale app_frontend=7```

Have fun with Traefik and container orchestration tools! 
