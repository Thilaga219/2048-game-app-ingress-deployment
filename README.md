# 2048-game-app-ingress-deployment
Deploying a game based application using ingress 
Pre-requisites:
1. kubectl - A command line tool for working with kubernetes. for more information, see Installing or updating kubectl.
2. eksctl -  A command line tool for working with EKS clusters that automates many individual tasks. For more information, see Installing or updating.
3. Get Access ID and secret Access keys from 
4. awscli - A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI in the AWS Command Line Interface User Guide
5. Creation of EKS
`eksctl create cluster --name app-cluster --region us-east-1 --fargate`
<img width="1624" height="784" alt="image" src="https://github.com/user-attachments/assets/7a20dfd5-8dfc-4096-9e70-6d0f80f8a08e" />
<img width="1918" height="427" alt="image" src="https://github.com/user-attachments/assets/d3961117-cc40-4c2d-a243-2fbe38e7edd0" />
<img width="1906" height="793" alt="image" src="https://github.com/user-attachments/assets/02b515b4-91d7-49be-b9a6-fdb7fddff519" />
<img width="1906" height="811" alt="image" src="https://github.com/user-attachments/assets/b082bb10-9110-43e0-af38-816b085674ed" />

<img width="800" height="329" alt="image" src="https://github.com/user-attachments/assets/841a6614-caec-4c12-a0ad-cd9664a21abd" />

7. Configuration of kubectl for EKS
`aws eks update-kubeconfig --name app-cluster --region us-east-1`

## Steps for deploying
### Step 1: Create Fargate Profile & Namespace under profile
`eksctl create fargateprofile --cluster app-cluster --region us-east-1 --name alb-sample-app --namespace game-2048`

### Step 2: Deploy the deployment.yml,service.yml,ingress.yml  
```kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml,kubectl get pods -n game-2048 ```
<img width="1354" height="673" alt="image" src="https://github.com/user-attachments/assets/8e1197e9-b77a-4180-962f-72d5c440b5a0" />

### Step 3: setup alb add on
        - Configure IAM OIDC provider 
        `eksctl utils associate-iam-oidc-provider --cluster app-cluster --approve`
<img width="1365" height="675" alt="image" src="https://github.com/user-attachments/assets/284c60ba-479e-499a-8589-ac86e7801ec2" />

        - Download IAM policy (can refer using ALB controller documentation)
        `curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json`
        - create IAM Policy
        `aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json`
<img width="1359" height="495" alt="image" src="https://github.com/user-attachments/assets/c026e318-1c1f-4848-8397-90e59172789a" />
        - Create IAM Role
        `eksctl create iamserviceaccount --cluster app-cluster --namespace kube-system --name aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn arn:aws:iam::191701055897:policy/AWSLoadBalancerControllerIAMPolicy --approve --override-existing-serviceaccounts`
<img width="1357" height="484" alt="image" src="https://github.com/user-attachments/assets/ab196bb7-d6aa-4298-9f14-64b35054ac76" />

        - **Deploy ALB controller** :
            - going to use heml chart for creating ALB controller
              - Add helm repo
              `helm repo add eks https://aws.github.io/eks-charts`
              - Update helm repo
              `helm repo update eks`
              - install helm charts
              `helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=app-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=us-east-1 --set vpcId=vpc-039c04c17b79cd091`
<img width="1357" height="484" alt="image" src="https://github.com/user-attachments/assets/3b4aa8ca-66ae-4873-82f1-794243fb6129" />

        - verify that the deployments are running.
        `kubectl get deployment -n kube-system aws-load-balancer-controller`
<img width="1905" height="936" alt="image" src="https://github.com/user-attachments/assets/bc995ec2-a96a-412a-af37-23624106423b" />

<img width="1846" height="736" alt="image" src="https://github.com/user-attachments/assets/473fb39f-dea5-426d-b4cb-3edb1443c9de" />
<img width="800" height="52" alt="image" src="https://github.com/user-attachments/assets/d99a3fd6-e7ec-4b15-aebf-ad964bbd0740" />
<img width="800" height="73" alt="image" src="https://github.com/user-attachments/assets/60a85b4a-16da-4a7b-bba8-3b8cc84bc1ad" />
<img width="1332" height="238" alt="image" src="https://github.com/user-attachments/assets/166fad23-db02-45f9-9501-4f7700eb9afc" />
<img width="800" height="196" alt="image" src="https://github.com/user-attachments/assets/4abae3b9-505b-48c6-a21f-709bdef5d084" />



