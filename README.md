# Coworking Space Service Extension
The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space. This service follows a microservice pattern and the APIs are split into distinct services that can be deployed and managed independently of one another.

The API provides business analysts basic analytics data on user activity in the service. The application they provide you functions as expected locally and you are expected to help build a pipeline to deploy it in Kubernetes.

## Dependencies
### AWS CLI
Ensure you have the necessary IAM permissions to create an EKS cluster. To verify your IAM identity with the following command:

```
aws sts get-caller-identity
```

If you encounter credential errors, you'll need to configure your AWS settings. To configure AWS CLI, on the terminal:

```
aws configure
```

Remeber to install `eksctl` if it does not built in: [Guideline](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html).

### Local Environment
1. Python Environment - run Python 3.6+ applications and install Python dependencies via `pip`
2. `Docker CLI` - build and run Docker images locally
3. `kubectl` - run commands against a Kubernetes cluster

### Remote Resources
1. `AWS CodeBuild` - build Docker images remotely
2. AWS ECR - host Docker images
3. `Kubernetes Environment` with `AWS EKS` - run applications in k8s
4. `AWS CloudWatch` - monitor activity and logs in EKS
5. `GitHub` - pull and clone code

## Setup

### 1. Create an EKS Cluster

#### Create an EKS Cluster
Install eksctl and use it to create an EKS cluster. It's as simple as running a single command to create a cluster:

```
eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name my-nodes --node-type t3.small --nodes 1 --nodes-min 1 --nodes-max 2
```

#### Update the Kubeconfig
After creating an EKS cluster, execute the command below to update the context in the local Kubeconfig file. This file allows configuring access to clusters, and should have at least one active context. It contains the information about clusters, users, namespaces, and authentication mechanisms.

```
aws eks --region us-east-1 update-kubeconfig --name my-cluster
```

### 2. Configure a Database for the Service

Apply YAML configurations in the following order.

```
cd deployment
kubectl apply -f pvc.yaml
kubectl apply -f pv.yaml
kubectl apply -f postgresql-deployment.yaml
cd ..
```

#### Connecting via Port Forwarding

Before you expose your database, you will need to create a service, and then expose it using port-forwarding approach. Apply the YAML `postgresql-service` at directory `deployment` to create the service `postgresql-service`:

```
cd deployment
kubectl apply -f postgresql-service.yaml
cd ..
```

Set up port-forwarding to `postgresql-service`. The **powershell** command below opens up port forwarding from your local environment's port 5433 to the node's port 5432:

```
Start-Job {kubectl port-forward svc/postgresql-service 5433:5432}
```

### 3. Deploy the Analytics Application

#### Continuous Integration with CodeBuild

The purpose of this step is to provide a systematic approach to pushing the Docker image of the coworking application into Amazon ECR, using `buildspec.yaml`:

1. Create an `Amazon ECR` repository on your AWS console.
2. Create an `Amazon CodeBuild` project that is connected to this GitHub repository.

In `buildspec.yaml`, please change the variable `AWS_ECR_URL` in `env` to your newly-created Amazon ECR. **Now with each Git push, an new, updated image will be created automatically and push to the ECR**.

#### Deployment

Apply YAML configurations in the following order.

```
cd deployment
kubectl apply -f configmap.yaml
kubectl apply -f coworking.yaml
cd ..
```

To verify the deployment, please `curl` the External-IP of the corresponding service obtained from `kubectl get svc`:

```
curl http://<External-IP>:5153/api/reports/daily_usage
curl http://<External-IP>:5153/api/reports/user_visits
```

### 4. Setup CloudWatch Logging

1. Attach the `CloudWatchAgentServerPolicy` IAM policy to your worker nodes:

    ```
    aws iam attach-role-policy \
    --role-name my-worker-node-role \
    --policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy 
    ```

    Replace `my-worker-node-role` with your EKS cluster's Node Group's IAM role.
2. Use AWS CLI to install the Amazon CloudWatch Observability EKS add-on:

    ```
    aws eks create-addon --addon-name amazon-cloudwatch-observability --cluster-name my-cluster-name
    ```

    Replace `my-cluster-name` with your EKS cluster's name.

3. Trigger logging by accessing your application.

4. Open up CloudWatch Log groups page. You should see `aws/containerinsights/my-cluster-name/application` there.



