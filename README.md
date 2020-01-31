# Computer Vision in Minutes with Amazon Rekognition and the AWS Python SDK

## Getting Started

[Amazon Rekognition](https://aws.amazon.com/rekognition/) is a fully managed AI API that makes it easy to add image and video analysis to your applications. With Amazon Rekognition, you can identify objects, people, text, scenes, and activities in images and videos, as well as detect any inappropriate content.

In this tutorial, we'll cover how to go from a fresh AWS account to working example Python code to detect labels in an image. 

This tutorial ass

To try Rekognition, you can check out one of the two demos:
	- [Webcam capture, with javascript sample code](https://ai-service-demos.go-aws.com/rekognition)
	- [Upload picture or provide URL](https://console.aws.amazon.com/rekognition/home?region=us-east-1#/label-detection)

## 1. Account Signup

### For Students:

- Follow the instructions on the [AWS Educate signup page](https://www.awseducate.com/registration#INFO-Student).

*Note:* Students are eligible for [AWS Educate](https://aws.amazon.com/education/awseducate/), a program dedicated to getting students, educators, and more started with using cloud technologies. AWS Educate includes annual credits, access to exclusive educational materials, and more!

### For Non-students:

- Follow the instructions on the default [AWS signup page](https://portal.aws.amazon.com/billing/signup.)


## 2. Create an IAM User 

Services in AWS, such as Amazon Rekognition, require that you provide credentials when you access them. This is so that the service can determine whether you have permissions to access the resources owned by that service. The console requires your password. You can create access keys for your AWS account to access the AWS CLI or API. However, we don't recommend that you access AWS by using the credentials for your AWS account. Instead, we recommend that you:

## 3. Local CLI and SDK Installation + Setup

### Install the SDK

- Install `boto3` (the AWS Python SDK) with `pip`.

```bash
$ pip install boto3
```

### Create an access key for the user you created in *Create an IAM User*.

- Sign in to the AWS Management Console and open the IAM console at https://console.aws.amazon.com/iam/.

- In the navigation pane, choose *Users*.

- Choose the name of the user you created in _Create an IAM User_ (previous step).

- Choose the *Security credentials* tab.

- Choose *Create access key*. Then choose Download .csv file to save the access key ID and secret access key to a CSV file on your computer. Store the file in a secure location. You will not have access to the secret access key again after this dialog box closes. After you have downloaded the CSV file, choose Close.

### Configure your local credentials
- On your computer, open a terminal and navigate to your home directory with `cd`.

- Create an `.aws` directory. On Unix-based systems, such as Linux or macOS, this is in the following location:

```bash
~/.aws
```

On Windows, this is in the following location:

```bash
%HOMEPATH%\.aws
```

In the .aws directory, create a new file named `credentials`.

Open the credentials CSV file that you created in step 2 and copy its contents into the credentials file using the following format:

```
[default]
aws_access_key_id = your_access_key_id
aws_secret_access_key = your_secret_access_key
```

_Substitute your acccess key ID and secret access key for your_access_key_id and your_secret_access_key._

- Save the `credentials` file and delete the CSV file.

## 4. Application Code

Rekognition has a large number of functionalities including:

- Labels Detection
- Faces Detection
- Faces Comparison
- Faces Indexing
- Faces Search
- Text Detection


For this tutorial we'll be using the *Detect Labels* method, but if you're interested in the other functions, you can find sample code for those [available here](https://gist.github.com/alexcasalboni/0f21a1889f09760f8981b643326730ff).

## Write Python code

- Create a new python file called `script.py` and paste the following code inside it.

```python3
import boto3

BUCKET = "amazon-rekognition"
KEY = "test.jpg"

def detect_labels(bucket, key, max_labels=10, min_confidence=90, region="eu-west-1"):
	rekognition = boto3.client("rekognition", region)
	response = rekognition.detect_labels(
		Image={
			"S3Object": {
				"Bucket": bucket,
				"Name": key,
			}
		},
		MaxLabels=max_labels,
		MinConfidence=min_confidence,
	)
	return response['Labels']


for label in detect_labels(BUCKET, KEY):
	print "{Name} - {Confidence}%".format(**label)

"""
	Expected output:
	People - 99.2436447144%
	Person - 99.2436447144%
	Human - 99.2351226807%
	Clothing - 96.7797698975%
	Suit - 96.7797698975%
"""
```

- Save the file.

### Run your code

- Open a terminal and `cd` into the directory where your `script.py` file resides.

- In your terminal:

`$ python detect-labels.py`

Congrats!

### Other Resources

- [Setup for CLI and Other SDKs](https://docs.aws.amazon.com/rekognition/latest/dg/setup-awscli-sdk.html)
- [AWS Rekognition Docs](https://docs.aws.amazon.com/rekognition/index.html)
- [Other AI Services/APIs](https://aws.amazon.com/machine-learning/ai-services/)

