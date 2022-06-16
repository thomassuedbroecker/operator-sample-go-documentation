# Overview

### Sample Go project

The [Operator Sample Go](https://github.com/IBM/operator-sample-go) project contains Kubernetes operator samples that demonstrate best practices of how to develop operators with [Golang](https://go.dev/), [Operator SDK](https://sdk.operatorframework.io/) including [Kubebuilder](https://github.com/kubernetes-sigs/kubebuilder) and the [Operator Lifecycle Manager Framework](https://operatorframework.io/).

The project contains two operators and four applications:

#### Application Operator

The application operator deploys and manages a front-end micro-service which provides a simple web UI.  The micro-service provides a single HTTP enpoint:
    
>**/hello** - Renders a Hello greeting to the names persisted in the corresponding database.
    
The application operator is responsible for the following tasks:

#### Deployment of Kubernetes resources:
* Deployment to manage front-end Pods, Service, Roles/Role Bindings, Cronjob (to launch auto-scaling decision logic)
* Database (a Custom Resource reconciled by the database operator, which creates the schema)
#### Auto-pilot operational tasks:
* Auto-scaling of pods according to how many times the /hello endpoint has been invoked

The application operator can be found in the [operator-application](https://github.com/IBM/operator-sample-go/tree/main/operator-application) folder.

#### Database Operator

The database operator relates to the backend database which provides a very simple file based database, included in this project.  It was created purely to help demonstrate the capabilities of operators.

The database implements a cluster where one pod is the 'leader' to which applications can perform reads and writes.  Further pods are 'followers' providing high availability and scalability.  Followers synchronize the data from the leader, but 
only provide read capability.  Each pod of the database cluster stores its data to a persistent volume.
    
The database is very opinionated and only supports a single use case - storing a list of first & last names, persisted in a JSON file.  Therefore only a single API is provided:
    
>**/persons** - GET/POST/PUT/DELETE a Person JSON object (other APIs help the pods elect the leader, and synchronize the data)
    
The database operator is responsible for the following tasks:

#### Deployment of Kubernetes resources:
* StatefulSet to manage stateful Pods and their binding to Persistent Volume Claims, Roles/Role Bindings, Service, Cronjob (to launch auto-backup logic)
#### Auto-pilot operational tasks:
* Immediate or scheduled backup of the database to one or more long term repositories (e.g. Cloud Object Storage)

The database operator can be found in the [operator-database](https://github.com/IBM/operator-sample-go/tree/main/operator-database) folder.


#### Assosiated Applications & Scripts

The repo provides the following applications, used by the operators

* [simple-microservice](https://github.com/IBM/operator-sample-go/tree/main/simple-microservice) - The front end web application, written in Java using Quarkus
* [database-service](https://github.com/IBM/operator-sample-go/tree/main/database-service) - The simple database application deployed by the database operator, written in Java using Quarkus
* [operator-database-backup](https://github.com/IBM/operator-sample-go/tree/main/operator-database-backup) - A Go application to query the database and upload the data to cloud object storage.  This container is launched on a schedule by the database operator
* [operator-application-scaler](https://github.com/IBM/operator-sample-go/tree/main/operator-application-scaler) - A Go application used to make autoscaling decisions for the front-end.  It queries Prometheus metrics exposed by the simple-microservice, and if necessary, modifies the custom resource which defines the size of the front-end deployment.  This container is launched on a schedule by the application operator

In addition, the repo includes:

* [scripts](https://github.com/IBM/operator-sample-go/tree/main/scripts) - Automation to verify workstation prerequisites, build all container images and deploy to a Kubernetes or OpenShift cluster.  Alternatively, the scripts can deploy pre-built 'golden' container images.