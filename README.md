# K8s_assignment

Part 1: Build the Application
Objective
To build a Java Servlet-based web application and generate a WAR file using Maven.

Description
This project is a simple Java Servlet application packaged as a WAR file. Maven is used as the build tool to compile the source code and create a deployable artifact that can run on a Tomcat server.

Build Command
mvn clean package
After successful execution, the WAR file is generated inside the target/ directory.

ls target
You should see:

docker-java-sample-webapp.war
How to Run
mvn clean package
ls target
Part 2: Dockerize the Application
Objective
To containerize the Java web application using a Tomcat Docker image.

Description
The generated WAR file is copied into a Tomcat container. The default Tomcat web applications are removed, and the custom WAR file is deployed as ROOT.war so it loads at the base URL.

Dockerfile
FROM tomcat:9-jdk17

RUN rm -rf /usr/local/tomcat/webapps/*

COPY target/docker-java-sample-webapp.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080

CMD ["catalina.sh", "run"]
Build Docker Image
If using Minikube locally, configure Docker to use Minikube’s Docker daemon:

eval $(minikube docker-env)
Build the image:

docker build -t tomcat-k8s-app .
Verify the image:

docker images
How to Run
eval $(minikube docker-env)
docker build -t tomcat-k8s-app .
docker images
Part 3: Deploy the Application to Kubernetes
Objective
To deploy the Dockerized application into a Kubernetes cluster using a Deployment and expose it using a NodePort Service.

Deployment Configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: tomcat-app
  template:
    metadata:
      labels:
        app: tomcat-app
    spec:
      containers:
      - name: tomcat-container
        image: tomcat-k8s-app
        imagePullPolicy: Never
        ports:
        - containerPort: 8080
Service Configuration
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
spec:
  type: NodePort
  selector:
    app: tomcat-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30008
Apply the Configuration
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
Verify the resources:

kubectl get pods
kubectl get svc
Pods should be in Running state and the service should expose port 30008.

How to Run
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods
kubectl get svc
Part 4: Access the Application
Objective
To access the deployed application through a browser.

Description
The NodePort service exposes the application externally on port 30008. Traffic flows from the NodePort to the Service, then to one of the running Pods, and finally to the Tomcat container serving the application.

Access Methods
Using Minikube command:

minikube service tomcat-service
Or manually:

minikube ip
Open the following URL in the browser:

http://<minikube-ip>:30008
The application should display the servlet response.

How to Run
minikube service tomcat-service
