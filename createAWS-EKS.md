# Download AWS CLI, eksctl, and kubctl (https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)

## AWS CLI
* Download AWS CLI
```
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```
* Configure your aws cli : (https://docs.google.com/document/d/1npYnTTGLfU76RzfTIvo3Iun0crM0-Y55EKQjKNiQTGM/edit#heading=h.vcfw9guhp991)

```
aws configure
aws_access_key_id = AK*************
aws_secret_access_key = DAR**************
```
* Edit your credentials to add the federated accounts
````
vi .aws/credentials
````

Add the following lines:
````
[SharedAccount]
role_arn = arn:aws:iam::555574965555:role/SC-France-BeLux
source_profile=default
````

* Test your AWS Cli :
````
aws ec2 describe-instances --profile SharedAccount --region eu-west-3
````

## Install EKSCTL & Kubectl

https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html#install-eksctl

https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html#eksctl-gs-install-kubectl
