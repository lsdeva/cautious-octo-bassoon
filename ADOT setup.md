## ADOT setup - manual 

Metrics Collection: Collect application, Kubernetes, and pod-level metrics.
Customizable: Highly configurable via OpenTelemetry collector pipelines.
Multi-backend Support: Can send data to multiple observability tools.

Organizations looking for an open-source, vendor-neutral solution for observability that supports multiple tools and tracing beyond just CloudWatch.


###
https://aws.amazon.com/blogs/containers/introducing-amazon-cloudwatch-container-insights-for-amazon-eks-fargate-using-aws-distro-for-opentelemetry/

###
required role and fargate profile creation
podexecution role - use existing 

#### namespaces
```bash
kubectl create namespace fargate-container-insights
kubectl create namespace golang
```

#### fargate profiles 
```bash
eksctl create fargateprofile \
  --cluster my-eks-cluster \
  --name fargate-container-insights \
  --namespace fargate-container-insights

eksctl create fargateprofile \
  --cluster my-eks-cluster \
  --name applications \
  --namespace golang
```

###

The ADOT Collector requires IAM permissions to send performance log events to CloudWatch. This is done by associating a Kubernetes service account with an IAM role using the IAM Roles for Service Accounts (IRSA) feature supported by EKS.
```bash
eksctl create iamserviceaccount \
--cluster=my-eks-cluster \
--region=ap-southeast-1 \
--name=adot-collector \
--namespace=fargate-container-insights \
--role-name=EKS-Fargate-ADOT-ServiceAccount-Role \
--attach-policy-arn=arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy \
--approve
```
###
 deploy the ADOT Collector as a Kubernetes StatefulSet using the following deployment manifest
 ```bash 
kubectl apply -f adot-StatefulSet.yaml
```

validation-
```bash
kubectl get statefulsets -n fargate-container-insights
kubectl get pods -n fargate-container-insights
```

deploy sample web application
```bash
kubectl create namespace golang
kubectl apply -f adot-sample-app-deploy.yaml
```
validate -
```bash
kubectl get deployments -n golang
```
