Infra Project:
  -VPC / subnets
  - load balance
  - DB
  - Web / App servers
  - S3
  - DNS
  - cdn
  - Iam Roles
  - Monitoring tools

  Sequence:
  - Repositories
  - s3 backend
  - vpc
  - web/app/db
  - iam roles

  Jira:
  - Ticketing/ bug tracking
  - Agile 
  - Agile boards - organise the work: 
    Scrum / Kanban

  Sprint meetings: - discuss last sprint
        - what went well / did not go well
        - work to be completed in the next sprint
        - create a sprint workflow / prioritizing work


  ===========================================================================================
Terraform:
- iac = manage and provision infra.
   - statefile - list of all resources
  - cloud agnostic = can be used with any / multiple cloud = aws,gcp,alibaba, kubernetes,azure = api
benefits - speed / consistency / repeatable / reduce errors
vendor - hashicorp = terraform, packer, vault
- download the binary = running terraform locally = running terraform commands from a local environment/ workstation / pipeline 
= Run terraform in the cloud = terraform cloud = enterprise / subscription
= Where you download binary = backend = where you run terraform operations

how it works:
- Configuration files / manifest files - .tf extension / .tf.json( )
= HCL - GO

Workflow: main.tf / bootcamp30.tf
        - invoke terraform = download provider plugins
terraform init: provider plugin / modules / set or initialise working dir = the dir that contains terraform manifest files

terraform validate: syntax / consistency
terraform fmt: syntax is correct
terraform plan : create an execution plan / blue print
           = credentials are required to generate a plan
terraform apply : apply the changes to reach desired state
terraform destroy: destroy terraform managed resources / ask for confirmation


===============================
Terraform blocks: 9 top level blocks

1. Terraform setting block:

terraform {
  required_version = "~> 1.0"         1.1.4/5/6/7   1.2/3/4/5 1.1.4/5/6/7
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
  backend "s3" {
    bucket = "terraform-mylandmark"
    key    = "prod/terraform.tfstate"
    region = "us-west-2"
    dynamodb_table = "terraform-lock"
  }
}

2. Provider block: plugin /api

provider "aws" {
  #profile = "default" # AWS Credentials Profile configured on your local desktop terminal  $HOME/.aws/credentials
  region  = "us-west-2"
}

3. Resource block:
  resource "aws_instance" "bootcamp31" {
     #ami           = "ami-0e5b6b6a9f3db6db8" # Amazon Linux
     ami = data.aws_ami.ubuntu.id
     instance_type = var.instance_type[1]
     delete_on_termination = true

     tags = {
      Name = local.name
     }
    }

4. Variables block / inputs

variable "instance_type" {
  description = "EC2 Instance Type"
  type = list(string)
  default = ["t2.micro", "t2.medium"]
}

5. Output blocks:

output "public_ip" {
  description = "ec2 instance public ip"
  value = aws_instance.inst1.arn
}

6. local value blocks:

  locals {
    name = "${var.app_name}-${var.environment}"
  }
  jenkins-production

7. Data sources:
   data "aws_ami" "ubuntu" {
     most_recent = true
     owners = ["self"]

     filter {
      name = "name"
      value = ["packer-docker"]
     }
   }

8. Modules block:

  module "ec2" {
    source = "./my_instance"
    version = "1.0.1"

    instance_type = var.instance_type
  }

9. moved blocks

moved {
  from = "aws_instance.bootcamp30 "
  to = "aws_instance.bootcamp31 "
}

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
K8S:
  a)Core concepts: 
    - definitions
    - architecture - master / slave / nodes
  b) Workload /scheduling
    a)Requests / limits  = cpu/memory = nodes
    - QoS = 1. Best Effort = no request/ no limits = first to be terminated / low priority
            2. Burstable = requests != limits  = second to be terminated
            3. Guaranteed = requests = limits
    b) Nodeselector
    c) Nodeaffinity / podaffinity
  c) Taints / tolerations - marks a node as unschedulable
  d) Draining - make it unavailable / take it offline
  e) Deployment strategies - rolling update / recreate / canary / blue-green
  f) services - cluster ip/ nodeport/ LB/ headless / externalname / ingress
  g) security - rbac - authentication / authorization = roles/clusterroles -permissions / namespace
              - network policies - isolate workloads / pods

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Helmfile:

#########
# Setup #
#########

# Install helmfile
https://helmfile.readthedocs.io/en/latest/
# Install helmfile diff plugin 
helm plugin install https://github.com/databus23/helm-diff
# Install helm git plugin
helm plugin install https://github.com/aslafy-z/helm-git --version 0.11.1


##############################
# Defining a simple Helmfile #
##############################

1. Create a file called helmfile.yaml
==========================================
repositories
- name: prometheus
    url: https://prometheus-community.github.io/helm-charts
  - name: helloworld
    url: git+https://github.com/Kenmakhanu/helmfile@helloworld?ref=main&sparse=0

releases:
  - name: prometheus
    namespace: prometheus
    chart: prometheus/prometheus
    #installed: true
    set:
      - name: rbac.create
        value: false
=======================================================================
2. Run the following command to install the chart 
   
   helmfile apply --wait
      or
   helmfile --interactive apply --wait   

3. If the file name is different then use the command below.

    helmfile --file helmfile-prometheus.yaml apply --wait

4. To uninstall the charts, you can use the destroy command
   
   helmfile --interactive destroy --wait

helmfile --interactive --file helmfile-prometheus-rbac.yaml apply --wait

helmfile --interactive --file helmfile-prometheus-rbac.yaml destroy


helmfile destroy


+++++++++++++++++++++++++++++++++++++++++++++++++++
argo cd:

# install ArgoCD in k8s
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# access ArgoCD UI
kubectl get svc -n argocd
kubectl port-forward svc/argocd-server 8080:443 -n argocd

# login with admin user and below token (as in documentation):
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo

# you can change and delete init password

# Switch to argodc namespace
# Modify the application.yaml with your git repository and apply to the cluster.
kubectl apply -f application.yaml

# Apply the secret in your cluster. In place of password, use token
kubectl apply -f secrets.yaml

# Push your manifest files to your repo


a) Push deployment and service to your repository
deployment.yaml
================
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  replicas: 2
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nanajanashia/argocd-app:1.2
        ports:
        - containerPort: 8080

service.yaml
=============
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080

b) Apply the application.yaml and secret.yaml to the cluster

application.yaml
================
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/yourrepo.git
    targetRevision: HEAD
    path: dev
  destination: 
    server: https://kubernetes.default.svc
    namespace: myapp

  syncPolicy:
    syncOptions:
    - CreateNamespace=true

    automated:
      selfHeal: true
      prune: true

secret.yaml
============
apiVersion: v1
kind: Secret
metadata:
  name: argocd-repo
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: https://github.com/yourrepo.git
  password: <your-token>
  username: yourusername
 





























