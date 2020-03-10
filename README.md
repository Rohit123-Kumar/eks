# eks
add iam role 
select eks
add role name
create vpc and security group using cloudformation
open eks documentation there we willm find a link in s3 to create this
https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-11-15/amazon-eks-vpc-private-subnets.yaml
or
https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-06-05/amazon-eks-vpc-sample.yaml

to create node --- create a stack through this template   
add clustername 
add vpc 
add min and max requirement
https://amazon-eks.s3-us-west-2.amazonaws.com/1.10.3/2018-06-05/amazon-eks-nodegroup.yaml





to install kubectl
curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl
curl -o kubectl.sha256 https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl.sha256
openssl sha1 -sha256 kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
kubectl version --short --client
mkdir ~/.kube
touch ~/.kube/config-eks
export KUBECONFIG=~/.kube/config-eks
open tis file and edit this 
Replace the API-SERVER-ENDPOINT placeholder with the API server endpoint obtained from the cluster detail page.
Replace the CA-DATA placeholder with the certificate authority data obtained from the cluster detail page.
Replace the CLUSTER-NAME placeholder with the name of the Amazon EKS cluster.
Replace the PROFILE-NAME placeholder with the name of your AWS credentials profile from the ~/.aws/credentials file (typically, default).
profile name is aws configure name

apiVersion: v1
clusters:
- cluster:
    server: API-SERVER-ENDPOINT ---copy from cluster
    certificate-authority-data: CA-DATA --copy from cluster
  name: kubernetes ..................same as cluster name
contexts:
- context:
    cluster: kubernetes ................same as clusetr name
    user: aws
  name: aws
current-context: aws
kind: Config
preferences: {}
users:
- name: aws
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      command: heptio-authenticator-aws replace it by aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - "CLUSTER-NAME..............same as clusetr name"
      env:
        - name: AWS_PROFILE
          value: "PROFILE-NAME"






to install iam autheniticator
curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/aws-iam-authenticator
chmod +x ./aws-iam-authenticator
 mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$PATH:$HOME/bin
aws-iam-authenticator version

aws configure --profile addanyname
enter access key
enter secret key
enter defalut region
defalut output
cat /root/.aws/credentials to see config aws is added or not


to install awscli
curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
 apt install unzip python
 unzip awscli-bundle.zip
 aws -versions

run any command like aws s3 ls to check the aws cli is working or not
after adding aws config only aws cli will work

####to add node to a cluster
create auth.yml file 
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: ARN-ROLE repalce this arn role with the node output arn
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes

kubectl apply -f auth.yaml  to run this file
