# Install Jenkins on local kubernetes cluster

#### Create a new namespace jenkins

```
kubectl create ns jenkins
```

#### Create Jenkins deployment file
create a jenkins-deployment.yaml file in the “jenkins” namespace we created in this section above.

This deployment file is defining a Deployment as indicated by the kind field.

The Deployment specifies a single replica. This ensures one and only one instance will be maintained by the Replication Controller in the event of failure.

The container image name is jenkins and version is 2.32.2

The list of ports specified within the spec are a list of ports to expose from the container on the Pods IP address.

Jenkins running on (http) port 8080.

The Pod exposes the port 8080 of the jenkins container.

The volumeMounts section of the file creates a Persistent Volume. This volume is mounted within the container at the path /var/jenkins_home and so modifications to data within /var/jenkins_home are written to the volume. The role of a persistent volume is to store basic Jenkins data and preserve it beyond the lifetime of a pod.


#### Deploy Jenkins
To create the deployment execute:

```
$ kubectl create -f jenkins-deployment.yaml -n jenkins
```

The command also instructs the system to install Jenkins within the jenkins namespace.

##### To validate that creating the deployment was successful you can invoke:

```
$ kubectl get deployments -n jenkins
```
##### Grant access to Jenkins service

We have a Jenkins instance deployed but it is still not accessible. The Jenkins Pod has been assigned an IP address that is internal to the Kubernetes cluster. It’s possible to log into the Kubernetes Node and access Jenkins from there but that’s not a very useful way to access the service.

To make Jenkins accessible outside the Kubernetes cluster the Pod needs to be exposed as a Service. A Service is an abstraction that exposes Jenkins to the wider network. It allows us to maintain a persistent connection to the pod regardless of the changes in the cluster. With a local deployment, this means creating a NodePort service type. A NodePort service type exposes a service on a port on each node in the cluster. The service is accessed through the Node IP address and the service nodePort. A simple service is defined here:

This service file is defining a Service as indicated by the kind field.

The Service is of type NodePort. Other options are ClusterIP (only accessible within the cluster) and LoadBalancer (IP address assigned by a cloud provider e.g. AWS Elastic IP).

The list of ports specified within the spec is a list of ports exposed by this service.

The port is the port that will be exposed by the service.

The target port is the port to access the Pods targeted by this service. A port name may also be specified.

The selector specifies the selection criteria for the Pods targeted by this service.

##### To create the service execute:

```
$ kubectl create -f jenkins-service.yaml -n jenkins
```

##### To validate that creating the service was successful you can run:

```
$ kubectl get services -n jenkins

NAME       TYPE        CLUSTER-IP       EXTERNAL-IP    PORT(S)           AGE
jenkins    NodePort    10.100.36.255    <none>         8080:32413/TCP    33m
```

##### Access Jenkins dashboard

So now we have created a deployment and service, how do we access Jenkins?

From the output above we can see that the service has been exposed on port 32664. We also know that because the service is of type NodeType the service will route requests made to any node on this port to the Jenkins pod. All that’s left for us is to determine the IP address of the minikube VM. Minikube have made this really simple by including a specific command that outputs the IP address of the running cluster:

  ```
$ ip a
192.168.118.99
```
Now we can access the Jenkins instance at 192.168.118.99:32413/

To access Jenkins, you initially need to enter your credentials. The default username for new installations is admin. The password can be obtained in several ways. This example uses the Jenkins deployment pod name.

To find the name of the pod, enter the following command:

```
$ kubectl get pods -n jenkins
```

Once you locate the name of the pod, use it to access the pod’s logs.

```
$ kubectl logs <pod_name> -n jenkins
The password is at the end of the log formatted as a long alphanumeric string:

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required.
An admin user has been created and a password generated.
Please use the following password to proceed to installation:

94b73ef6578c4b4692a157f768b2cfef
```

This may also be found at:
/var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
You have successfully installed Jenkins on your Kubernetes cluster and can use it to create new and efficient development pipelines.
