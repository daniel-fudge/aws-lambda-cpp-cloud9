# aws-lambda-cpp-cloud9
This repo builds an AWS C++ Lambda function in the Cloud9 environment. It repeats the same locally build Lambda function in this [repo](https://github.com/daniel-fudge/aws-lambda-cpp-local-build).   
A video walk through of the video can be found [here](https://youtu.be/olO5ORrq1cU).

## Install Some Dependencies
Note the AWS Linux AMI uses `yum` instead of `apt-get`. The version of CMake is also very old so we need to manually download a new version and install.
```bash
cd ~
sudo yum update -y
sudo yum install libcurl-devel -y
wget https://cmake.org/files/v3.19/cmake-3.19.5.tar.gz
tar -xvzf cmake-3.19.5.tar.gz 
cd cmake-3.19.5
sudo ./bootstrap
sudo make
sudo make install
```

## Build the AWS C++ SDK
These steps install the C++ SDK into the `~/install` directory and clones all the directories into `~/environment`. This can obviously be altered if desired.    
We are also only installing the `core` package. Other packages may be required for more involved Lambda functions. 
```bash
mkdir ~/install
cd ~
git clone --recurse-submodules https://github.com/aws/aws-sdk-cpp.git
cd aws-sdk-cpp
mkdir build
cd build
cmake .. -DBUILD_ONLY="core" \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=OFF \
  -DCUSTOM_MEMORY_MANAGEMENT=OFF \
  -DCMAKE_INSTALL_PREFIX=~/install \
  -DENABLE_UNITY_BUILD=ON
make
make install
```

## Build the AWS C++ Lambda Runtime
```bash
cd ~
git clone https://github.com/awslabs/aws-lambda-cpp-runtime.git
cd aws-lambda-cpp-runtime
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=OFF \
  -DCMAKE_INSTALL_PREFIX=~/install
make
make install
```

## Build the Actual C++ Lambda Function
```bash
cd ~
git clone https://github.com/daniel-fudge/aws-lambda-cpp-cloud9.git
cd aws-lambda-cpp-cloud9
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=~/install
make
make aws-lambda-package-demo
```

## Make IAM Role for the Lambda Function
Note this is the same steps as in the local repo and there is no need to repeat if you have already made the role. You will have to copy the role ARM from the console.   

We need to create and IAM role that the can be attached to the lambda function when it is deployed. Note the rights in this role may need to be expanded for more complex Lambda functions.  
First create a JSON file that defines the required permissions as shown below.
```JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"Service": ["lambda.amazonaws.com"]},
      "Action": "sts:AssumeRole"
    }
  ]
}
```
Then create the IAM role in the CLI as shown below.
```bash
aws iam create-role --role-name lambda-demo --assume-role-policy-document file://trust-policy.json
```
The output of the above command will include the ARN of the new role. You must copy this ARN. It will be required when you deploy the Lambda function. It will most like have the form `arn:aws:iam::<your AWS account number>:role/lambda-demo`.   

Next attached the `AWSLambdaBasicExecutionRole` policy to the new role to allow the Lambda function to write to CloudWatch Logs. This is performed with the following CloudWatch command.
```bash 
aws iam attach-role-policy --role-name lambda-demo --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

## Deploy 
We can now deploy the lambda function with the following CLI command.
```bash
aws lambda create-function --function-name demo-cpp-cloud9 \
  --role <specify role arn from previous step here> \
  --runtime provided --timeout 15 --memory-size 128 \
  --handler demo-cpp-cloud9 --zip-file fileb://demo.zip
```

## Test
```bash
aws lambda invoke --function-name demo-cpp-cloud9 --cli-binary-format raw-in-base64-out --payload '{"location": "somewhere"}' output.json
```

## References
- [Cloud9 Video](https://youtu.be/olO5ORrq1cU)
- [Local C++ Lambda Repo](https://github.com/daniel-fudge/aws-lambda-cpp-local-build)
- [CMake Downloads](https://cmake.org/download/)
- [Introducing the C Lambda Runtime](https://aws.amazon.com/blogs/compute/introducing-the-c-lambda-runtime/)
- [C++ Sample Lab](https://github.com/awslabs/aws-lambda-cpp)
- [AWS CLI - Invoke Lambda](https://docs.aws.amazon.com/cli/latest/reference/lambda/invoke.html#examples)
- [AWS CLI - Payload Error](https://stackoverflow.com/questions/60310607/amazon-aws-cli-not-allowing-valid-json-in-payload-parameter)
- [AWS Lambda Runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
- [AWS EC2 Pricing](https://aws.amazon.com/ec2/pricing/on-demand/)
