# CLO835 Final Project - Group 4

## Group Members:
- Nishit Rathod - Seneca ID: 133411223
- Omkumar Sanjay Goswami - Seneca ID: 140472234
- Meet Brahmbhatt - Seneca ID: 136557220
- Harjot Singh - Seneca ID: 127800233

This repository contains the deployment instructions for a two-tier web application on Amazon EKS. The application uses Docker containers hosted on Amazon ECR and includes an application server and a MySQL database. Follow the steps below to set up and deploy the application.

## Prerequisites

- AWS CLI installed and configured
- Docker installed
- eksctl installed
- kubectl installed
- hey installed (for load testing)

## Docker Setup

1. **Authenticate Docker to the Amazon ECR registry:**

    ```bash
    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 937274605158.dkr.ecr.us-east-1.amazonaws.com
    ```

2. **Pull the Docker images:**

    ```bash
    docker pull 937274605158.dkr.ecr.us-east-1.amazonaws.com/docker-prod-containers-application:latest
    docker pull 937274605158.dkr.ecr.us-east-1.amazonaws.com/docker-prod-containers-database:latest
    ```

3. **Create a Docker network:**

    ```bash
    docker network create my-network
    docker network ls
    ```

4. **Run the MySQL database container:**

    ```bash
    docker run -d --name my-database-container --network my-network -p 3306:3306 -e MYSQL_ROOT_PASSWORD=passwors 937274605158.dkr.ecr.us-east-1.amazonaws.com/docker-prod-containers-database:latest
    ```

5. **Run the application container:**

    ```bash
    docker run -d --name my-application-container --network my-network -p 81:81 \
    -e DBHOST=my-database-container \
    -e DBPORT=3306 \
    -e DBUSER=root \
    -e DBPWD=passwors \
    -e DATABASE=employees \
    -e BACKGROUND_IMAGE=https://privatebucketclo835.s3.amazonaws.com/blue.jpg \
    -e GROUP_NAME="GROUP4" \
    -e AWS_ACCESS_KEY_ID= \
    -e AWS_SECRET_ACCESS_KEY= \
    -e AWS_SESSION_TOKEN= \
    937274605158.dkr.ecr.us-east-1.amazonaws.com/docker-prod-containers-application:latest
    ```

6. **Access the application:**

    Visit `http://3.94.98.51:81/` in your browser.

## Kubernetes Setup

1. **Navigate to the project directory:**

    ```bash
    cd CLO835-FinalProject-Group4/
    cd manifests/
    ```

2. **Create the EKS cluster:**

    ```bash
    eksctl create cluster -f eks_config.yaml
    kubectl get nodes
    ```

3. **Install the AWS EBS CSI driver:**

    ```bash
    export AWS_REGION=us-east-1
    eksctl create addon --name aws-ebs-csi-driver --cluster clo835-final --service-account-role-arn arn:aws:iam::937274605158:role/LabRole --force
    eksctl get addons --cluster clo835-final
    ```

4. **Apply the Kubernetes resources:**

    ```bash
    kubectl apply -f namespace.yaml

    cd database/

    kubectl apply -f storageclass.yaml
    kubectl apply -f pvc.yaml
    kubectl apply -f secrets.yaml
    kubectl apply -f service-account.yaml
    kubectl apply -f service.yaml
    kubectl apply -f statefulset.yaml
    kubectl apply -f deployment.yaml

    kubectl get all -n final

    cd ../application/

    kubectl apply -f config-map.yaml
    kubectl apply -f deployment.yaml
    kubectl apply -f service.yaml
    kubectl apply -f hpa.yaml
    kubectl apply -f role.yaml
    kubectl apply -f role-binding.yaml

    kubectl get all -n final
    kubectl describe service application-service -n final
    ```

5. **Check Persistent Volumes:**

    ```bash
    kubectl get pv
    kubectl get pvc -n final
    ```

6. **Interact with the MySQL database:**

    ```bash
    kubectl exec -it mysql-0 -n final -- mysql -u root -p
    USE employees;
    SELECT * FROM employee;
    INSERT INTO employee VALUES ('5','Aayush','Rathod','Smile','local');
    kubectl delete pod mysql-0 -n final
    ```

7. **Check the service and load test:**

    ```bash
    kubectl get service application-service -n final
    hey -h
    hey -n 1000 -c 100 http://af5ed11e39f26478ba1c45bd7db07191-527009310.us-east-1.elb.amazonaws.com
    ```

## Conclusion

This project demonstrates the deployment of a scalable, containerized application on Amazon EKS with persistent storage, load balancing, and auto-scaling capabilities. 
