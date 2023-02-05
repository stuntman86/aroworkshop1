
 # For every resource that you deploy on ARO (project/app etc) please append your student name number to it. For example if the documentation mentions to create a parksmap app , student 1 should create parksmap1 , student 2 parksmap2 etc                                        
                                       
                                         
# Lab 1 : Deploy a simple application on ARO                                           
                                   

This **Lab** is intended to give you a hands on introduction to using OpenShift from the perspective of a developer.

* Topics which this workshop will cover include:

* Using the OpenShift command line client and web console.

* Deploying an application using a pre-existing container image.

* Working with application labels to identify component parts.

* Scaling up your application in order to handle web traffic.

* Exposing your application to users outside of the cluster.

* Viewing and working with logs generated by your application.

* Accessing your application container and interacting with it.

* Architecture Overview of the ParksMap Application

This lab introduces you to the architecture of the ParksMap application used throughout this workshop, to get a better understanding of the things you’ll be doing from a developer perspective. ParksMap is a polyglot geo-spatial data visualization application built using the microservices architecture and is composed of a set of services which are developed using different programming languages and frameworks.

![image](https://user-images.githubusercontent.com/32516987/216835216-21049e46-6b7c-4b81-92ec-7508da7ef9bc.png)


The main service is a web application which has a server-side component in charge of aggregating the geo-spatial APIs provided by multiple independent backend services and a client-side component in JavaScript that is responsible for visualizing the geo-spatial data on the map. The client-side component which runs in your browser communicates with the server-side via WebSockets protocol in order to update the map in real-time.

There will be a set of independent backend services deployed that will provide different mapping and geo-spatial information. The set of available backend services that provide geo-spatial information are:

WorldWide National Parks

Major League Baseball Stadiums in North America

The original source code for this application is located here.

The server-side component of the ParksMap web application acts as a communication gateway to all the available backends. These backends will be dynamically discovered by using service discovery mechanisms provided by OpenShift which will be discussed in more details in the following labs.

In this lab, we’re going to deploy the web component of the ParksMap application which is also called parksmap and uses OpenShift’s service discovery mechanism to discover the backend services deployed and shows their data on the map.

![image](https://user-images.githubusercontent.com/32516987/216835267-1c33e8c5-80b3-4ba5-8a19-6bacd298b2ae.png)


Exercise: Deploying your First Image

Let’s start by doing the simplest thing possible - get a plain old Docker-formatted image to run on Azure Red Hat OpenShift. This is incredibly simple to do. With OpenShift it can be done directly from the web console.

If you’re no longer on the Topology view in the Developer perspective, return there now. Click Container Image to open a dialog that will allow you to specify the information for the image you want to deploy.

![image](https://user-images.githubusercontent.com/32516987/216835273-0c62a5b1-f1b1-44ad-a633-64f700057e78.png)




In the Image Name field, copy and paste the following into the box :

> quay.io/openshiftroadshow/parksmap:latest

OpenShift will then go out to the container registry specified and interrogate the image.

Your screen will end up looking something like this:

![image](https://user-images.githubusercontent.com/32516987/216835292-6fb2f896-9de1-4693-b1b6-15cde2152a37.png)




In Runtime Icon you can select the icon to use in OpenShift Topology View for the app. You can leave the default OpenShift icon, or select one from the list.

Make sure to have the correct values in:

* Application Name : workshop

* Name : parksmap

Ensure **Deployment** is selected from Resource section.

Un-check the checkbox next to **Create a route to the application.** For learning purposes, we will create a Route for the application later in the workshop.

At the bottom of the page, click Labels in the Advanced Options section and add some labels to better identify this deployment later. Labels will help us identify and filter components in the web console and in the command line.

We will add 3 labels. After you enter the name=value pair for each label, press tab or de-focus with mouse before typing the next. First the name to be given to the application.

> app=workshop

Next we apply the name of this deployment

> component=parksmap

And last, the role this component plays as part of the overall app

> role=frontend

![image](https://user-images.githubusercontent.com/32516987/216835436-35923116-8c34-48b0-8411-970f78210ec5.png)


Next, click the blue Create button. You will be directed to the Topology page, where you should see the visualization for the parksmap deployment config in the workshop application.

![image](https://user-images.githubusercontent.com/32516987/216835452-420083f4-d1dd-417e-a33e-d746be48983b.png)




These few steps are the only ones you need to run to get a container image deployed on OpenShift. This should work with any container image that follows best practices, such as defining an EXPOSE port, not needing to run specifically as the root user or other user name, and a single non-exiting CMD to execute on start.

Background: Containers and Pods

Before we start digging in, we need to understand how containers and Pods are related. We will not be covering the background on these technologies in this lab but if you have questions please inform the instructor. Instead, we will dive right in and start using them.

In OpenShift, the smallest deployable unit is a Pod. A Pod is a group of one or more OCI containers deployed together and guaranteed to be on the same host. From the official OpenShift documentation:

” Each Pod has its own IP address, therefore owning its entire port space, and containers within pods can share storage. Pods can be “tagged” with one or more labels, which are then used to select and manage groups of pods in a single operation. Pods can contain multiple OCI containers. The general idea is for a Pod to contain a “main process” and any auxiliary services you want to run along with that process. Examples of containers you might put in a Pod are, an Apache HTTPD server, a log analyzer, and a file service to help manage uploaded files. “

Exercise: Examining the Pod

If you click on the parksmap entry in the Topology view, you will see some information about that deployment config. The Resources tab may be displayed by default. If so, click on the Details tab. On that panel, you will see that there is a single Pod that was created by your actions.

![image](https://user-images.githubusercontent.com/32516987/216835468-6dd833ca-39e4-4f4f-a265-ca0da771daea.png)


You can also get a list of all the Pods created within your Project, by navigating to Workloads → Pods in the Administrator perspective of the web console.

![image](https://user-images.githubusercontent.com/32516987/216835474-20880643-07c6-4fc4-b69e-8bf16fdee169.png)



This Pod contains a single container, which happens to be the parksmap application - a simple Spring Boot/Java application.

You can also examine Pods from the command line:

> oc get pods

You should see output that looks similar to:

![image](https://user-images.githubusercontent.com/32516987/216835485-0061f5b0-71ba-4c89-96ba-b32b8d04c52f.png)


The above output lists all of the Pods in the current Project, including the Pod name, state, restarts, and uptime. Once you have a Pod’s name, you can get more information about the Pod using the oc get command. To make the output readable, I suggest changing the output type to YAML using the following syntax:

> oc get pod parksmap-65c4f8b676-k5gkk -o yaml

You should see something like the following output (which has been truncated due to space considerations of this workshop manual):

![image](https://user-images.githubusercontent.com/32516987/216835493-90d03fea-87bf-4471-a24e-3a14ef454f38.png)


The web interface also shows a lot of the same information on the Pod details page. If you click on the name of the Pod, you will find the details page. You can also get there by clicking on the parksmap deployment config on the Topology page, selecting Resources, and then clicking the Pod name.


![image](https://user-images.githubusercontent.com/32516987/216835501-8f505f1d-2fff-43e4-b9ea-adcac398f42a.png)

![image](https://user-images.githubusercontent.com/32516987/216835508-80b0c037-b9c3-45f8-946a-d7a03a585aaf.png)


Getting the parksmap image running may take a little while to complete. Each OpenShift node that is asked to run the image has to pull (download) it, if the node does not already have it cached locally. You can check on the status of the image download and deployment in the Pod details page, or from the command line with the oc get pods command that you used before.

**Background: Services**

Services provide a convenient abstraction layer inside OpenShift to find a group of similar Pods. They also act as an internal proxy/load balancer between those Pods and anything else that needs to access them from inside the OpenShift environment. For example, if you needed more parksmap instances to handle the load, you could spin up more Pods. OpenShift automatically maps them as endpoints to the Service, and the incoming requests would not notice anything different except that the Service was now doing a better job handling the requests.

When you asked OpenShift to run the image, it automatically created a Service for you. Remember that services are an internal construct. They are not available to the “outside world”, or anything that is outside the OpenShift environment. That’s okay, as you will learn later.

The way that a Service maps to a set of Pods is via a system of Labels and Selectors. Services are assigned a fixed IP address and many ports and protocols can be mapped.

There is a lot more information about Services, including the YAML format to make one by hand, in the official documentation.

Now that we understand the basics of what a Service is, let’s take a look at the Service that was created for the image that we just deployed. In order to view the Services defined in your Project, enter in the following command:

> oc get services

You should see something like this :

![image](https://user-images.githubusercontent.com/32516987/216835528-427cadc0-154c-4313-9663-0adca93472df.png)


In the above output, we can see that we have a Service named parksmap with an IP/Port combination of 172.30.22.209/8080TCP. Your IP address may be different, as each Service receives a unique IP address upon creation. Service IPs are fixed and never change for the life of the Service.

In the Developer perspective from the Topology view, service information is available by clicking the parksmap deployment config, then Resources, and then you should see the parksmap entry in the Services section.

![image](https://user-images.githubusercontent.com/32516987/216835536-f0642403-a04f-49ab-8097-9fbf35b32e2b.png)


You can also get more detailed information about a Service by using the following command to display the data in YAML:

> oc get service parksmap -o yaml

Which should provide an output like this :

![image](https://user-images.githubusercontent.com/32516987/216835547-eef875a7-2bcf-4611-93d0-168290ce0a88.png)



Take note of the selector stanza. Remember it.

Alternatively, you can use the web console to view information about the Service by clicking on it from the previous screen.

![image](https://user-images.githubusercontent.com/32516987/216835569-3d198030-489d-44c1-9179-867df60b6c8c.png)


It is also of interest to view the YAML of the Pod to understand how OpenShift wires components together. For example, run the following command to get the name of your parksmap Pod:

> oc get pods

You should get something like this :

![image](https://user-images.githubusercontent.com/32516987/216835574-b8d886b7-f124-4d1a-9eb7-f68f6ddc5ea7.png)


Now you can view the detailed data for your Pod with the following command:

> oc get pod parksmap-65c4f8b676-k5gkk -o yaml

Under the metadata section you should see the following:

![image](https://user-images.githubusercontent.com/32516987/216835590-ad036a98-6788-46e9-a07c-b8b350446cf4.png)


The Service has selector stanza that refers to deploymentconfig=parksmap.

The Pod has multiple Labels:

> app=parksmap

> deploymentconfig=parksmap

Labels are just key/value pairs. Any Pod in this Project that has a Label that matches the Selector will be associated with the Service. To see this in action, issue the following command:

> oc describe service parksmap

![image](https://user-images.githubusercontent.com/32516987/216835603-102e1d6c-0894-45c6-b0ec-06b57cbb33c8.png)



Now that you have succesfully deployed an application on ARO and you have seen how you can observe and disect it , you are ready to go to the 2nd part of the lab where we can see how an application running on ARO can get scaled up and down depending on demand and “revive” in case of a failed pod.

# Lab 2: Scaling up and down / Self Healing

**Background: Deployments and ReplicaSets** 

While **Services** provide routing and load balancing for **Pods**, which may go in and out of existence, **ReplicaSet (RS)** and **ReplicationController (RC)** are used to specify and then ensure the desired number of **Pods** (replicas) are in existence. For example, if you always want your application server to be scaled to **3 Pods (instances)**, a **ReplicaSet** is needed. Without an **RS**, any **Pods** that are killed or somehow die/exit are not automatically restarted. **ReplicaSets** and **ReplicationController** are how **Azure Red Hat OpenShift** "self heals" and while **Deployments** control **ReplicaSets**, **ReplicationController** here are controlled by **DeploymentConfigs**.

In Kubernetes, a Deployment (D) defines how something should be deployed. In almost all cases, you will end up using the Pod, Service, ReplicaSet and Deployment resources together. And, in almost all of those cases, OpenShift will create all of them for you.

There are some edge cases where you might want some Pods and an RS without a D or a Service, and others, so feel free to ask us about them after the labs.


## Exercise: Exploring Deployment-related Objects

Now that we know the background of what a ReplicaSet and Deployment are, we can explore how they work and are related. Take a look at the Deployment (D) that was created for you when you told OpenShift to stand up the parksmap image:

> oc get deployment

![image](https://user-images.githubusercontent.com/32516987/216836256-d51041d6-2a1d-45af-8c8a-a2ec754a984e.png)

To get more details, we can look into the ReplicaSet (RS).

Take a look at the ReplicaSet (RS) that was created for you when you told OpenShift to stand up the parksmap image:

> oc get rs

![image](https://user-images.githubusercontent.com/32516987/216836292-0b22e3fe-7540-4e41-83c2-37eb01c033af.png)

This lets us know that, right now, we expect one Pod to be deployed (Desired), and we have one Pod actually deployed (Current). By changing the desired number, we can tell OpenShift that we want more or less Pods.

OpenShift’s HorizontalPodAutoscaler effectively monitors the CPU usage of a set of instances and then manipulates the RCs accordingly.

## Exercise: Scaling the Application

Let’s scale our parksmap "application" up to 2 instances. We can do this with the scale command. You could also do this by incrementing the Desired Count in the OpenShift web console. Pick one of these methods; it’s your choice.

> oc scale --replicas=2 deployment/parksmap

You can also scale up to two pods in the Developer Perspective. From the Topology view, first click the parksmap deployment config and select the Details tab:

![image](https://user-images.githubusercontent.com/32516987/216836353-ef48a60f-d0a7-478f-bbb9-82f164d3a2d1.png)

Next, click the ^ icon next to the Pod visualization to scale up to 2 pods.

![image](https://user-images.githubusercontent.com/32516987/216836367-3edac46e-3e32-4765-805f-da9a64c93c1a.png)

To verify that we changed the number of replicas, issue the following command:

> oc get rs

![image](https://user-images.githubusercontent.com/32516987/216836397-3d0656b3-bfe8-480c-b7b7-d8d4494f49e7.png)

You can see that we now have 2 replicas. Let’s verify the number of pods with the oc get pods command:

> oc get pods


![image](https://user-images.githubusercontent.com/32516987/216836419-149a50a1-dc9b-428f-aed3-03e91b9dd8d4.png)

And lastly, let’s verify that the Service that we learned about in the previous lab accurately reflects two endpoints:

> oc describe svc parksmap

You will get soemthing similar to this:

![image](https://user-images.githubusercontent.com/32516987/216836538-4abc5524-c201-43cc-aaf9-f6a2a9507713.png)

Another way to look at a Service's endpoints is with the following:

> oc get endpoints parksmap

which will result in something like this :

![image](https://user-images.githubusercontent.com/32516987/216836575-353eb823-0ff2-4ec5-9da0-d59d6afdd258.png)

Your IP addresses will likely be different, as each pod receives a unique IP within the OpenShift environment. The endpoint list is a quick way to see how many pods are behind a service.

You can also see that both Pods are running in the Developer Perspective:      

![image](https://user-images.githubusercontent.com/32516987/216836600-76e1e45d-e24e-4212-82e9-4bdbdb5d788d.png)

Overall, that’s how simple it is to scale an application (Pods in a Service). Application scaling can happen extremely quickly because OpenShift is just launching new instances of an existing image, especially if that image is already cached on the node.

## Application "Self Healing"

Because OpenShift’s RSs are constantly monitoring to see that the desired number of Pods actually are running, you might also expect that OpenShift will "fix" the situation if it is ever not right. You would be correct!

Since we have two Pods running right now, let’s see what happens if we "accidentally" kill one. Run the oc get pods command again, and choose a Pod name. Then, do the following:

> oc delete pod parksmap-65c4f8b676-k5gkk && oc get pods

![image](https://user-images.githubusercontent.com/32516987/216836636-f703d0ad-3440-4013-b7a6-96eff6a2b5d0.png)


Did you notice anything? One container has been deleted, and there’s a new container already being created.

Also, the names of the Pods are slightly changed. That’s because OpenShift almost immediately detected that the current state (1 Pod) didn’t match the desired state (2 Pods), and it fixed it by scheduling another Pod.

Additionally, OpenShift provides rudimentary capabilities around checking the liveness and/or readiness of application instances. If the basic checks are insufficient, OpenShift also allows you to run a command inside the container in order to perform the check. That command could be a complicated script that uses any installed language.

Based on these health checks, if OpenShift decided that our parksmap application instance wasn’t alive, it would kill the instance and then restart it, always ensuring that the desired number of replicas was in place.

## Exercise : Scale Down

Before we continue, go ahead and scale your application down to a single instance. Feel free to do this using whatever method you like.


# Exposing your Application to the Outside World


In this lab, we’re going to make our application visible to the end users, so they can access it.

![image](https://user-images.githubusercontent.com/32516987/216836689-1b5f6169-9d32-40e1-966f-1a71f468c70d.png)


## Routes

While Services provide internal abstraction and load balancing within an OpenShift environment, sometimes clients (users, systems, devices, etc.) outside of OpenShift need to access an application. The way that external clients are able to access applications running in OpenShift is through the OpenShift routing layer. And the data object behind that is a Route.

The default OpenShift router (HAProxy) uses the HTTP header of the incoming request to determine where to proxy the connection. You can optionally define security, such as TLS, for the Route. If you want your Services, and, by extension, your Pods, to be accessible from the outside world, you need to create a Route.

## Exercise : Create a Route

You may remember that when we deployed the parksmap application, we un-checked the checkbox to create a Route. Normally it would have been created for us automatically. Fortunately, creating a Route is a pretty straight-forward process. You simply expose the Service via the command line. Or, via the Administrator Perspective, just click Networking → Routes and then the Create Route button.

Insert parksmap in Name field.

From Service field, select parksmap. For Target Port, select 8080.

In Security section, check Secure route. Select Edge from TLS Termination list.

Leave all other fields blank and click Create:


![image](https://user-images.githubusercontent.com/32516987/216836732-8063ded2-c8e3-4116-9edd-b28ff0c190f5.png)


![image](https://user-images.githubusercontent.com/32516987/216836738-cb5e3711-8a76-411b-85be-808810b55a23.png)

When creating a Route, some other options can be provided, like the hostname and path for the Route or the other TLS configurations.

When using the command line, we can first verify that we don’t already have any existing Routes:

> oc get routes

> No resources found

Now lets get the service name we want to expose

> oc get services

![image](https://user-images.githubusercontent.com/32516987/216836767-677ac081-a6a3-458e-9b2d-8756a9ffb23a.png)


Once we know the Service name, creating a Route is a simple one-command task:


> oc create route edge parksmap --service=parksmap

> route.route.openshift.io/parksmap exposed

Verify the Route was created with the following command:

> oc get route

![image](https://user-images.githubusercontent.com/32516987/216836795-1876f013-551c-4414-975e-693ece390ca2.png)


You can also verify the Route in the Developer Perspective under the Resources tab for your parksmap deployment configuration. Also note that there is a decorator icon on the parksmap visualization now. If you click that, it will open the URL for your Route in a browser.

![image](https://user-images.githubusercontent.com/32516987/216836805-7392f2e9-0b61-4158-8c20-3dbde8d858ce.png)


This application is now available at the URL shown in the Developer Perspective. Click the link and you will see it.

> At first time, the Browser will ask permission to get your position. This is needed by the Frontend app to center the world map to your location, if you don’t allow it, it will just use a default location.

![image](https://user-images.githubusercontent.com/32516987/216836825-85748a63-0136-450c-86f0-944bd683b3df.png)


# Exploring ARO's Logging Capabilities

OpenShift provides some convenient mechanisms for viewing application logs. First and foremost is the ability to examine a Pod's logs directly from the web console or via the command line.

## Exercise: Examining logs

Since we already deployed our application, we can take some time to examine its logs. In the Developer Perspective, from Topology view, click the parksmap entry and then the Resources tab. You should see a View Logs link next to the Pod entry.

![image](https://user-images.githubusercontent.com/32516987/216836886-d6a2a8b2-b26e-41da-a908-92d6bb24b69b.png)


Click the View Logs link and you should see a nice view of the Pod's logs:

![image](https://user-images.githubusercontent.com/32516987/216836892-0a2dd1ac-3f76-4d11-87d8-361cc48b1da0.png)


You also have the option of viewing logs from the command line. Get the name of your Pod:

> oc get pods

![image](https://user-images.githubusercontent.com/32516987/216836915-fe7c7374-8362-4096-a470-3b5ebe2ccbd8.png)

And then use the logs command to view this Pod's logs:

> oc logs parksmap-1-hx0kv


You will see all of the application logs scroll on your screen:

![image](https://user-images.githubusercontent.com/32516987/216836941-73f73302-8c3f-4a1e-a1fc-6462e0a2acf6.png)


#Lab 2 : ARO Internals (if you would like to continue experimenting on different stuff of your choice on ARO instead , please go ahead)

We will use an application named OStoy that will help us with this Lab.

### About OSToy
OSToy is a simple Node.js application that we will deploy to Azure Red Hat OpenShift. It is used to help us explore the functionality of Kubernetes. This application has a user interface which you can:

* write messages to the log (stdout / stderr)
* intentionally crash the application to view self-healing
* toggle a liveness probe and monitor OpenShift behavior
* read config maps, secrets, and env variables
* if connected to shared storage, read and write files
* check network connectivity, intra-cluster DNS, and intra-communication with an included microservice
* increase the load to view automatic scaling of the pods to handle the load (via the Horizontal Pod Autoscaler)

### OSToy Application Diagram

![image](https://user-images.githubusercontent.com/32516987/216837205-69efefdb-c3e7-448f-9a48-4cc3f5392929.png)


### Familiarization with the Application UI

1. Shows the pod name that served your browser the page.
2. Home: The main page of the application where you can perform some of the functions listed which we will explore.
3. Persistent Storage: Allows us to write data to the persistent volume bound to this application.
4. Config Maps: Shows the contents of configmaps available to the application and the key:value pairs.
5. Secrets: Shows the contents of secrets available to the application and the key:value pairs.
6. ENV Variables: Shows the environment variables available to the application.
7. Auto Scaling: Explore the Horizontal Pod Autoscaler to see how increased loads are handled.
8. Networking: Tools to illustrate networking within the application.
9. About: Shows some more information about the application.

![image](https://user-images.githubusercontent.com/32516987/216837264-2d5d56b4-7522-4d2b-9a79-78f5a2144ef5.png)


### Learn more about the application

To learn more, click on the “About” menu item on the left once we deploy the app.

![image](https://user-images.githubusercontent.com/32516987/216837290-86149fdd-95b9-4efa-aec8-bbf5cd8baf93.png)

## Application Deployment

### Create new project

Create a new project called "ostoy" in your cluster

> oc new-project ostoy

You can do the same from the Web UI if you prefer : 

![image](https://user-images.githubusercontent.com/32516987/216837381-c63e2e07-4f9d-449c-8db4-8624e0fb1ce6.png)


### Deploy the backend microservice

In your terminal deploy the microservice using the following command:

> oc apply -f https://raw.githubusercontent.com/microsoft/aroworkshop/master/yaml/ostoy-microservice-deployment.yaml

You should get the following response: 



> oc apply -f https://raw.githubusercontent.com/microsoft/aroworkshop/master/yaml/ostoy-microservice-deployment.yaml
deployment.apps/ostoy-microservice created
service/ostoy-microservice-svc created



### Deploy the frontent service

This deployment contains the node.js frontend for our application along with a few other Kubernetes objects.

> oc apply -f https://raw.githubusercontent.com/microsoft/aroworkshop/master/yaml/ostoy-fe-deployment.yaml

You should see all objects created successfully

> oc apply -f https://raw.githubusercontent.com/microsoft/aroworkshop/master/yaml/ostoy-fe-deployment.yaml
persistentvolumeclaim/ostoy-pvc created
deployment.apps/ostoy-frontend created
service/ostoy-frontend-svc created
route.route.openshift.io/ostoy-route created
configmap/ostoy-configmap-env created
secret/ostoy-secret-env created
configmap/ostoy-configmap-files created
secret/ostoy-secret created

### Get Route

Get the route so that we can access the application via:



> oc get route

You should get something like this: 

> NAME           HOST/PORT                                                      PATH      SERVICES              PORT      TERMINATION   WILDCARD
ostoy-route   ostoy-route-ostoy.apps.abcd1234.westus2.aroapp.io             ostoy-frontend-svc   <all>                   None
