<!--
Deploying Microservices in Julia
Adam Michael
-->

Pathfinder uses several different routing engines to find the optimal route for your application. From a software perspective, the most interesting is a webserver written in [Julia](http://julialang.org/) that uses linear programming.

Julia is a young language with a focus on technical computing, which made it a perfect candidate to develop a linear programming algorithm with. Our Julia routing server runs significantly faster than our other servers (written in Java, Scala, Node.js and Ruby). However, the downside to a young language is that the ecosystem lacks a lot to be desired. In this post, we will describe the steps we at Pathfinder took to productionize a Julia microservice.

## Webservice
Choices for a Julia webserver are fairly limited at the moment. You can either use `HttpServer` or roll your own implementation. We used `HttpServer` which requires `MbedTLS`. You will also probably want the `JSON` library.

    julia -e 'Pkg.add("MbedTLS"); Pkg.add("HttpServer"); Pkg.add("JSON");'

To create a basic webserver, use the following code:

    # server.jl
    using HttpServer
    using JSON

    http = HttpHandler() do req::Request, res::Response
        request_dictionary = JSON.parse(bytestring(req.data))
        response = JSON.json(Dict("status" => "Got your message!"))
        return Response(response)
    end
    server = Server(http)
    run(server, 2929)

You can now start the server up with:

    julia server.jl

You may be asking, where is the routing logic? There is none! We believe strongly in small, atomic microservices that do one thing and do it well. This service just happens to return the string "Got your message!" very effectively.

## Containerization
We deploy all of our microservices using Kubernetes on the Google Container Engine (GKE). To do this, we need to build a Docker image for our service.

### Specifying dependencies
Julia has a mechanism for specifying dependencies using `REQUIRE` files. The `REQUIRE` file for our server looks like this

    julia 0.4

    HttpServer
    JSON
    MbedTLS

### Dockerfile
At this time, there is no standard Julia Dockerfile. So we wrote our own based on Ubuntu 14.04.

First you will need to indicate the parent Docker file.

    FROM ubuntu:14.04

Next you will list the maintainer.

    MAINTAINER Adam Michael <adam@ajmichael.net>

Next you will install Julia and all of its dependencies.

    RUN apt-get update && apt-get install -y wget python-software-properties software-properties-common libglfw2 libglfw-dev
    RUN add-apt-repository ppa:staticfloat/juliareleases && apt-get update
    RUN apt-get install -y build-essential cmake xorg-dev libglu1-mesa-dev git libgmp-dev
    RUN apt-get install -y julia
    RUN julia -e "Pkg.resolve()"

Next you will add the `REQUIRE` file that tells Julia which packages you will be using and then have Julia install them.

    ADD REQUIRE /.julia/v0.4/REQUIRE
    RUN julia -e "Pkg.resolve()"

Now it is time to add your code and run it.

    ADD server.jl /
    CMD ["julia", "/server.jl"]


You can find our complete Dockerfile for reference on [GitHub](https://github.com/CSSE497/pathfinder-routing/blob/master/linearprogramming/Dockerfile). Note that it has some additional dependencies that are specific to us.

### Building
You can build a Docker image with:

    docker build -t <name of image here>:0.0.1 .
    
Since we use GKE, our image names have to match the container registry where we push them, so our build command usually looks like:

    docker build -t beta.gcr.io/pathfinder-gcloud-project-name/server:0.0.1 .

### Running
You can run the container locally with Docker:

    docker run -p 2929:2929 -t beta.gcr.io/pathfinder-gcloud-project-name/server:0.0.1 .

If you use GKE to run your Docker containers you can push the image to Google Container Registry (GCR) with:

    gcloud docker push beta.gcr.io/pathfinder-gcloud-project-name/server:0.0.1

To run it on GKE, you will first need a container cluster which you can create with:

    gcloud container clusters create name-of-cluster --num-nodes 1

Then you will need to get auth access for that container cluster for Kubernetes:

    gcloud beta container clusters get-credentials name-of-cluster

Then you can run the image on the cluster with:

    kubectl run name-of-replication-controller --image=beta.gcr.io/pathfinder-gcloud-project-name/server:0.0.1 --port=2929

Then you will need to expose the port to the world with a load balancer:

    kubectl expose name-of-replication-controller --create-external-load-balancer=true --port=80 --target-port=2929

Your service is now live! You can get its external IP address by:

    $ kubectl get svc
    NAME         CLUSTER-IP       EXTERNAL-IP     PORT(S)   AGE
    kubernetes   10.183.240.1     <none>          443/TCP   7d
    your-svc     10.183.251.193   104.137.22.55   80/TCP    1d

You can then test out your service with:

    curl -i -H "Content-type: application/json" -X POST -d "{my:json}" http://104.137.22.55


Congratulations! You can now develop and release your own microservices with Julia!
