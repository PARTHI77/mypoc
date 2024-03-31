Install Jenkins with YAML files
This section describes how to use a set of YAML (Yet Another Markup Language) files to install Jenkins on a Kubernetes cluster. The YAML files are easily tracked, edited, and can be reused indefinitely.

Create Jenkins deployment file
Copy the contents here into your preferred text editor and create a jenkins-deployment.yaml file in the “jenkins” namespace we created in this section above.

This deployment file is defining a Deployment as indicated by the kind field.

The Deployment specifies a single replica. This ensures one and only one instance will be maintained by the Replication Controller in the event of failure.

The container image name is jenkins and version is 2.32.2

The list of ports specified within the spec are a list of ports to expose from the container on the Pods IP address.

Jenkins running on (http) port 8080.

The Pod exposes the port 8080 of the jenkins container.

The volumeMounts section of the file creates a Persistent Volume. This volume is mounted within the container at the path /var/jenkins_home and so modifications to data within /var/jenkins_home are written to the volume. The role of a persistent volume is to store basic Jenkins data and preserve it beyond the lifetime of a pod.

Exit and save the changes once you add the content to the Jenkins deployment file.

Deploy Jenkins
To create the deployment execute:

kubectl create -f jenkins-deployment.yaml -n jenkins
The command also instructs the system to install Jenkins within the jenkins namespace.

To validate that creating the deployment was successful you can invoke:

kubectl get deployments -n jenkins
Grant access to Jenkins service
We have a Jenkins instance deployed but it is still not accessible. The Jenkins Pod has been assigned an IP address that is internal to the Kubernetes cluster. It’s possible to log into the Kubernetes Node and access Jenkins from there but that’s not a very useful way to access the service.

To make Jenkins accessible outside the Kubernetes cluster the Pod needs to be exposed as a Service. A Service is an abstraction that exposes Jenkins to the wider network. It allows us to maintain a persistent connection to the pod regardless of the changes in the cluster. With a local deployment, this means creating a NodePort service type. A NodePort service type exposes a service on a port on each node in the cluster. The service is accessed through the Node IP address and the service nodePort. A simple service is defined here:

This service file is defining a Service as indicated by the kind field.

The Service is of type NodePort. Other options are ClusterIP (only accessible within the cluster) and LoadBalancer (IP address assigned by a cloud provider e.g. AWS Elastic IP).

The list of ports specified within the spec is a list of ports exposed by this service.

The port is the port that will be exposed by the service.

The target port is the port to access the Pods targeted by this service. A port name may also be specified.

The selector specifies the selection criteria for the Pods targeted by this service.

To create the service execute:

kubectl create -f jenkins-service.yaml -n jenkins
To validate that creating the service was successful you can run:

kubectl get services -n jenkins
NAME       TYPE        CLUSTER-IP       EXTERNAL-IP    PORT(S)           AGE
jenkins    NodePort    10.103.31.217    <none>         8080:32664/TCP    59s
Access Jenkins dashboard
So now we have created a deployment and service, how do we access Jenkins?

From the output above we can see that the service has been exposed on port 32664. We also know that because the service is of type NodeType the service will route requests made to any node on this port to the Jenkins pod. All that’s left for us is to determine the IP address of the minikube VM. Minikube have made this really simple by including a specific command that outputs the IP address of the running cluster:

minikube ip
192.168.99.100
Now we can access the Jenkins instance at 192.168.99.100:32664/

To access Jenkins, you initially need to enter your credentials. The default username for new installations is admin. The password can be obtained in several ways. This example uses the Jenkins deployment pod name.

To find the name of the pod, enter the following command:

kubectl get pods -n jenkins
Once you locate the name of the pod, use it to access the pod’s logs.

kubectl logs <pod_name> -n jenkins
The password is at the end of the log formatted as a long alphanumeric string:

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required.
An admin user has been created and a password generated.
Please use the following password to proceed to installation:

94b73ef6578c4b4692a157f768b2cfef

This may also be found at:
/var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
You have successfully installed Jenkins on your Kubernetes cluster and can use it to create new and efficient development pipelines.

Install Jenkins with Jenkins Operator
The Jenkins Operator is a Kubernetes native Operator which manages operations for Jenkins on Kubernetes.

It was built with immutability and declarative configuration as code in mind, to automate many of the manual tasks required to deploy and run Jenkins on Kubernetes.

Jenkins Operator is easy to install with applying just a few yaml manifests or with the use of Helm.

For instructions on installing Jenkins Operator on your Kubernetes cluster and deploying and configuring Jenkins there, see official documentation of Jenkins Operator.

Post-installation setup wizard
After downloading, installing and running Jenkins using one of the procedures above (except for installation with Jenkins Operator), the post-installation setup wizard begins.

This setup wizard takes you through a few quick "one-off" steps to unlock Jenkins, customize it with plugins and create the first administrator user through which you can continue accessing Jenkins.

Unlocking Jenkins
When you first access a new Jenkins instance, you are asked to unlock it using an automatically-generated password.

Browse to http://localhost:8080 (or whichever port you configured for Jenkins when installing it) and wait until the Unlock Jenkins page appears.

Unlock Jenkins page

From the Jenkins console log output, copy the automatically-generated alphanumeric password (between the 2 sets of asterisks).

Copying initial admin password
Note:

The command: sudo cat /var/lib/jenkins/secrets/initialAdminPassword will print the password at console.

If you are running Jenkins in Docker using the official jenkins/jenkins image you can use sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword to print the password in the console without having to exec into the container.

On the Unlock Jenkins page, paste this password into the Administrator password field and click Continue.
Note:

The Jenkins console log indicates the location (in the Jenkins home directory) where this password can also be obtained. This password must be entered in the setup wizard on new Jenkins installations before you can access Jenkins’s main UI. This password also serves as the default administrator account’s password (with username "admin") if you happen to skip the subsequent user-creation step in the setup wizard.

Customizing Jenkins with plugins
After unlocking Jenkins, the Customize Jenkins page appears. Here you can install any number of useful plugins as part of your initial setup.

Click one of the two options shown:

Install suggested plugins - to install the recommended set of plugins, which are based on most common use cases.

Select plugins to install - to choose which set of plugins to initially install. When you first access the plugin selection page, the suggested plugins are selected by default.

If you are not sure what plugins you need, choose Install suggested plugins. You can install (or remove) additional Jenkins plugins at a later point in time via the Manage Jenkins > Plugins page in Jenkins.
The setup wizard shows the progression of Jenkins being configured and your chosen set of Jenkins plugins being installed. This process may take a few minutes.

Creating the first administrator user
Finally, after customizing Jenkins with plugins, Jenkins asks you to create your first administrator user.

When the Create First Admin User page appears, specify the details for your administrator user in the respective fields and click Save and Finish.

When the Jenkins is ready page appears, click Start using Jenkins.
Notes:

This page may indicate Jenkins is almost ready! instead and if so, click Restart.

If the page does not automatically refresh after a minute, use your web browser to refresh the page manually.

If required, log in to Jenkins with the credentials of the user you just created and you are ready to start using Jenkins!
