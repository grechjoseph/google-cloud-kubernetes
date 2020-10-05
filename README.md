<h1>Deploying a Spring Boot Application to Google Cloud's Kubernetes</h1>

<h2>Contents</h2>
<ol>
    <li><a href="#prerequisites">Prerequisites</a></li>
    <li><a href="#create_cluster">Create Cluster</a></li>
    <li><a href="#preparing_application_for_deployment">Preparing Application for Deployment</a></li>
    <li><a href="#deploying_from_container_to_pods">Deploying From Container to Pods</a></li>
    <li><a href="#routing_traffic">Routing Traffic (Load Balancing)</a></li>
    <li><a href="#scaling">Scaling</a></li>
</ol>

<h2 id="prerequisites">Prerequisites</h2>
<ol>
    <li>This application ready on a git repo.</li>
    <li>Google Cloud access.</li>
    <li>Google Cloud Project created.</li>
</ol>

<h2 id="create_cluster">Create Cluster</h2>
<ol>
    <li>While on the Google Cloud Project selected, navigate to <b>Kubernetes Engine</b> -> <b>Clusters</b>.</li>
    <li>When the page is loaded, click <b>Create Cluster</b>. For this example, name it <b>spring-boot-cluster</b>.</li>
    <li>Location Type: Leave default (Zone).</li>
    <li>Master Version: Leave default.</li>
    <li>Number of Nodes: 3 (normally by default) with 3.75GB memory and 1vCPU each.</li>
    <li>NOTE: More Pools can be added. This is simply the default pool.</li>
</ol>

This create a Kubernetes Master, which includes the API Servcer, Scheduler, etcd, and ControllerManagement. This also create the three Nodes specified, each having the KubeProxy, Docker, and Kubelet installed.

<h2 id="preparing_application_for_deployment">Preparing Application for Deployment</h2>
<ol>
    <li>Once the cluster is created, open the Cloud Shell on the top right corner's button <b>Activate Cloud Shell</b>.</li>
    <li>Once Shell is loaded: <b>git clone {url to repository}</b>.</li>
    <li><b>cd {repository name}</b>.</li>
    <li><b>./mvnw clean install</b> or <b>mvn clean install</b>.</li>
    <li>Verify that the JAR file is create in folder <b>target</b>.</li>
    <li><b>java -jar /target/{jar name}</b> to check that the JAR file runs.</li>
</ol>

At this point, you can also test that the application ran is reachable:
<ol>
    <li>On the top right, select the <b>Web Preview</b> button.</li>
    <li>Specify port and path in URL to verify that the REST Controller works.</li>
</ol>

Once verified, continue with the below:
<ol>
    <li>Stop the JAR file from running (<b>Ctrl + C</b>).</li>
    <li>Create Docker image with command: <b>./mvnw com.google.cloud.tools:jib-maven-plugin:build -Dimage=gcr.io/$GOOGLE_CLOUD_PROJECT/spring-boot-application:v1</b>.</li>
    <li>This creates the docker images and uploads it to the Container Repository on your Google Cloud.</li>
    <li><b>gcr.io</b>: Google Container Repository.</li>
    <li><b>$GOOGLE_CLOUD_PROJECT</b>: Variable with Project ID.</li>
    <li><b>jib-maven-plugin</b>: plugin that builds docker file.</li>
    <li>Google Container Repository can be either hosted Externally or Internally. To access the one on your Google Cloud, navigate to <b>Container Registry</b> -> <b>Images</b>.</li>
    <li>Open the image created and verify its name and version.</li>
</ol>

<h2 id="deploying_from_container_to_pods">Deploying From Container to Pods</h2>
<ol>
    <li>In Cloud Shell, switch to the cluster using command line: <b>gcloud container clusters get-credentials {cluster name / spring-boot-cluster} --zone {cluster zone, default: us-central1-c}</b>.</li>
    <li><b>kubectl get pods</b> should be empty.</li>
    <li><b>kubectl get services</b> should show KUBERNETES only.</li>
    <li><b>kubectl get deployments</b> should be empty.</li>
    <li>Test the docker image: <b>docker run -ti --rm -p 8080:8080 {url to image, gcr.io/...}</b></li>
    <li>You can use <b>Web Preview</b> again to test.</li>
    <li>Deploy to cluster: <b>kubectl create deployment {application name} --image={url to image, gcr.io/...} --port=8080</b> (eg: <b>kubectl create deployment spring-boot-deployment --image=gcr.io/spring-boot-application-291608/spring-boot-application:v1 --port=8080</b>. NOTE: 8080 is the port to expose.</li>
    <li>Check: <b>kubectl get deployments</b> should now list the deployment.</li>
    <li>Check: <b>kubectl get pods</b> should now list one pod.</li>
    <li>Check: <b>kubectl get services</b> should still be empty.</li>
</ol>

<h2 id="routing_traffic">Routing Traffic (Load Balancing)</h2>
<ol>
    <li>Run command: <b>kubectl expose deployment spring-boot-deployment --type=LoadBalancer</b> or <b>kubectl expose deployment spring-boot-deployment --type=LoadBalancer --port=80 --target-port=8080</b> to set up port forwarding.</li>
    <li>Check: <b>kubectl get services</b> now shows the application with External IP being either <b>PENDING</b> or the <b>IP</b>.</li>
    <li>NOTE: Not included in these steps is the ability to create a DNS besides the IP.</li>
    <li>Service should now be accessible using the <b>External IP of the Load Balancer</b>.</li>
</ol>

<h2 id="scaling">Scaling</h2>
To scale the application to all three pods/instances:
<ol>
    <li>Run command: <b>kubectl scale deployment {deployment name} --replicas=3</b>.</li>
    <li>This should response with message <b>... scaled.</b></li>
    <li>Check: <b>kubectl get pods</b> should now list 3 pods instead of 1.</li>
    <li>Check: <b>kubectl get deployments</b> should now list the deployment with instances set to the <b>3</b>.</li>
    <li>NOTE: The Load Balancer will now automatically Load Balance between these three pods.</li>
</ol>
