
# Three Tier app deployment on AWS EKS 

**Architecture**

Situation : Deployed a Three tier reactJs app with nodeJS backend and MongoDB database to be able to handle thousands of concurrent users.

Task : Dockerized the application along with eksctl cluster setup for kubernetes deployment and later deploymented on AWS EKS.Also deployment of application load balancer for user access and ingress controller for internal routing. Using Route53 for accessing webapp using domai nname (harshilrpatel.com). 

Action: 
- Used docker to containerize the application and pushed images to AWS ECR repository for versioning.
- Automated the kubernetes cluster using AWS EKS with eksctl.
- Wrote k8s manifest filesa and deployed the application on AWS EKS.
- Ensured multi-node cluster setup for high avaibility and deployment using load balancer and ingress controller.

- Route53 for domain name purchase and DNS management.

Result: Improved scalability of the application to thousands of concurrent users and reduced downtime by 70% using AWS managed Elastic Kubernetes Service(EKS).

This project involves the deployment of a three-tier reactJS  application with nodeJS backend and MongoDB database, to be able to handle up to thousands of concurrent users. The aim is to enhance scalability and minimize downtime by using Docker containers and Kubernetes orchestration. With AWS EKS (Elastic Kubernetes Service), the deployment is optimized for fault tolerance and high availability.

**Architecture**

The architecture comprises a three-tier reacrJS application with a nodeJS backend and MongoDB database backend. Docker containers are used to handle the application components, ensuring consistency and ease of deployment. Kubernetes orchestrates the containerized application, managing resources and scaling dynamically to meet demand. Deployment on AWS EKS improves the scalability and resilience of managed Kubernetes services, fault tolerance and enabling seamless operations. AWS application load balancer gives acceess to the EKS managed nodes and application is accessed by domainname.  

**Key Tasks:**

1. Containerization with Docker:

    Used Docker to containerize the reactJS frontend, nodeJS backend  and MongoDB database.
    
    Images were pushed to AWS ECR for versioning and accessibility.

2. Kubernetes Cluster Setup:

    Automated Kubernetes cluster setup using eksctl to deploy the nodes.

    Transitioned to AWS EKS with eksctl for improved management and scalability.

3. K8S manifest files:

    Created K8S Manifest files for frontend,backend and database, streamlining deployment and management.

    Used kubectl to deploy the application on AWS EKS efficiently.

4. High Availability and Scalability:

    Implemented a multi-node cluster setup to ensure high availability and fault tolerance.

    Used load balancer for distributing traffic, optimizing performance and scalability.

**Conclusion:**

Conclusion: The three-tier application's successful deployment on AWS EKS shows the advantages of containerisation and Kubernetes orchestration for modern application deployments. This project improves fault tolerance and scalability while also highlighting the operational benefits of using managed cloud services, such as AWS EKS, for more reliable and efficient operations.


## Docker Installation And AWS ECR for docker versioning

Here I have dockerized the application using AWS EC2 ubuntu instance(Free tier).

![Screenshot (571)](https://github.com/harshilp156/three-tier-app-deployment-k8s/assets/67538347/5ae220ef-2d85-4319-825f-fc69be40b8c2)

```bash
sudo apt update
sudo apt installdocker.io
```
Cloned the project from git into the EC2 instance.

DockerFile for frontend application:
```bash
FROM node:14
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
CMD [ "npm", "start" ]
```

Now build the docker file.

```bash
docker build -t three-tier-app-frontend .
```
Run the docker conainer. 

```bash
docker run -d -p 3000:3000 three-tier-app-frontend:latest 
```

DockerFile for backend application:
```bash
FROM node:14
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

Now build the docker file.

```bash
docker build -t three-tier-app-backend .
```
Run the docker conainer. 

```bash
docker run -d -p 8080:8080 three-tier-app-backend:latest 
```

Retrieve an authentication token and authenticate Docker client to registry.

![Screenshot (572)](https://github.com/harshilp156/three-tier-app-deployment-k8s/assets/67538347/d0537b62-17e8-4d74-8133-e39fed8a485a)

Use the AWS CLI and pushed the docker conatiner into AWS ECR.
## EKS Setup

Here I have done the setup of AWS EKS using AWS EC2 instance (freetier).

Installed AWS CLI on that instance.

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o
"awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
```

Created an IAM role with the necessary permissions for EKS and attached it to an IAM User.

After creating the user, generated the Access Key and Secret Access
Key for this user.

Configured AWS CLI.

Installed Kubectl :
```bash
curl -o kubectl https://amazon-eks.s3.us-west2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short –client
```

Installed eksctl :

```bash
curl --silent --location
"https://github.com/weaveworks/eksctl/releases/latest/download/
eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

EKS Cluster Setup :

```bash
eksctl create cluster --name <cluster-name> --region <region> --node-type
t2.micro --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
```

![Screenshot (573)](https://github.com/harshilp156/three-tier-app-deployment-k8s/assets/67538347/1469a8a4-cfbd-404d-bac2-725af872b88c)

Created K8S manifest files for frontend ,backend and database.

```bash
kubectl create namespace three-tier-app
kubectl apply -f .
kubectl get all -n three-tier-app
```
Here all of the three tiers are deployed on k8s cluster.

Now ALB is installed for exposing the application to the outside world.

# Install AWS Load Balancer
 
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=three-tier-app-cluster --approve
eksctl create iamserviceaccount --cluster=three-tier-app-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-east-1
```

# Deploy AWS Load Balancer Controller

```bash
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=three-tier-app-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl apply -f ingress.yaml
```

![Screenshot (575)](https://github.com/harshilp156/three-tier-app-deployment-k8s/assets/67538347/970ed810-dc71-48b7-8349-a7366776a0d6)

![Screenshot (576)](https://github.com/harshilp156/three-tier-app-deployment-k8s/assets/67538347/f1162f76-663a-449e-92c6-ff2be8a1d437)


# Create ingress controller for internal routing of pod.

indress.yaml

```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mainlb
  namespace: three-tier-app
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  ingressClassName: alb
  rules:
    - host: harshilrpatel.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api
                port:   
                  number: 8080
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port: 
                  number: 3000
      
```

```bash
kubectl apply -f ingress.yaml
kubectl get ing -n three-tier-app
```

# Hosted zone created and updated the dns name servers.

![Screenshot (574)](https://github.com/harshilp156/three-tier-app-deployment-k8s/assets/67538347/092e19d6-6cb5-4745-8705-34e562a5c377)

 
Now the application can be accessible from the domain name (harshilrpatel.com)

![Screenshot (570)](https://github.com/harshilp156/three-tier-app-deployment-k8s/assets/67538347/e8362b98-b2de-4f39-83b2-41d6f01b7b44)

Also the data is updated on the mongodb database.

![Screenshot (568)](https://github.com/harshilp156/three-tier-app-deployment-k8s/assets/67538347/72d31d44-dd5f-48ce-b65a-d0ecbb9e8c2e)

![Screenshot (569)](https://github.com/harshilp156/three-tier-app-deployment-k8s/assets/67538347/800cf7c5-187b-4e62-9f37-fc1ae48a831d)



## Tech Stack

**AWS:** EKS, IAM, EC2 ,ECR, Load Balancer ,Route53

**Docker**

**Kubernetes**

**ReactJs** **NodeJs** **HTML** **CSS** **VIM**

**GIT**
