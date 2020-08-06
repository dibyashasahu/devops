# configure local machine

### pre-req setup

Installing awscli

    brew install awscli

generate private and public key if does not exist

    cd ~/.ssh
    ssh-keygen -t rsa


configure aws secrets

    export AWS_ACCESS_KEY=<>
    export AWS_SECRET_KEY=<>

### kubernates cluster creation


configure cluster name and bucket name

    bucket_name=sarojim-2020
    export KOPS_CLUSTER_NAME=imesh.k8s.local
    export KOPS_STATE_STORE=s3://sarojim-2020

installing kops

    brew install kops


if aws configuration is not done yet. we did everything but region

    AWS Access Key ID [None]: AccessKeyValue
    AWS Secret Access Key [None]: SecretAccessKeyValue
    Default region name [None]: us-east-1
    Default output format [None]:

Create bucket if does not exist

    aws s3api create-bucket \
    --bucket ${bucket_name} \
    --region us-east-1

enable versioning if new bucket

    aws s3api put-bucket-versioning --bucket ${bucket_name} --versioning-configuration Status=Enabled


create secret

    kops create secret --name imesh.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub


create cluster

    kops create cluster \
    --node-count=2 \
    --node-size=t2.micro \
    --zones=us-east-1a \
    --name=${KOPS_CLUSTER_NAME}

update cluster

    kops update cluster --yes

edit cluster

    kops edit cluster --name ${KOPS_CLUSTER_NAME}

    kops update cluster --name ${KOPS_CLUSTER_NAME} --yes
    kops validate cluster

Dashboard

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

get secrets

    kops get secrets kube --type secret -oplaintext
    ## answer: 2KEokTpGl4pVLbsFhazf3U4bxYgDkeD5

    kubectl cluster-info

    kubectl proxy --address 0.0.0.0 --accept-hosts '.*'

    http://localhost:8001/
    https://api-imesh-k8s-local-d8ok51-1298043401.us-east-1.elb.amazonaws.com/ui


    https://<kubernetes-master-hostname>/ui

Delete cluster

    kops delete cluster --name ${KOPS_CLUSTER_NAME} --yes
    
create environmemnt
    
    kubectl create namespace dev
    kubectl create namespace test
    kubectl create namespace prod
    kubectl get namespaces
    
 create deployment
    vi deployment.yaml
    
apiVersion: apps/v1
kind: Deployment
metadata:
 name: jenkins-deployment
spec:
 replicas: 3
 selector:
   matchLabels:
     app: jenkins
 template:
   metadata:
     labels:
       app: jenkins
   spec:
     containers:
     - name: jenkins
       image: jenkins/jenkins:lts
       ports:
       - containerPort: 8080

    
    kubectl create -f deployment.yaml --namespace test
    kubectl get deployments
    kubectl get deployments --namespace test
    kubectl get pods --namespace test
    vi service.yaml
    
    apiVersion: v1
kind: Service
metadata:
 name: jenkins
spec:
 type: NodePort
 ports:
   - port: 8080
     targetPort: 8080
     nodePort: 32000
 selector:
   app: jenkins

Expose the service for deployment
    kubectl create -f service.yaml --namespace test
    kubectl describe pods --namespace test
 provide 32000 in the inbound role, of node , check private dns
  kubectl logs jenkins-deployment-86b4cbc868-p2w5n --namespace test
  copy password, paste in jenkins 
    
