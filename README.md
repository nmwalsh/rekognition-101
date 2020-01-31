# Computer Vision in Minutes with Amazon Rekognition and the AWS Python SDK

## Getting Started

[Amazon Rekognition](https://aws.amazon.com/rekognition/) is a fully managed AI API that makes it easy to add image and video analysis to your applications. With Amazon Rekognition, you can identify objects, people, text, scenes, and activities in images and videos, as well as detect any inappropriate content.

In this tutorial, we'll cover how to go from a fresh AWS account to working example Python code to detect labels in an image. 

If you'd like to get a taste for what Rekognition can do, check out the live demos listed below:

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

- Navigate to [these instructions](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) and follow the method for *Creating an Administrator IAM User and Group (Console)*

- Add the following permissions for the new IAM user's role:

	- `AmazonRekognitionFullAccess`
	- `AmazonS3FullAccess`

*Note:* An IAM user with administrator permissions has unrestricted access to the AWS services in your account. For information about restricting access to Amazon Rekognition operations, see Using Identity-Based Policies (IAM Policies) for Amazon Rekognition. The code examples in this guide assume that you have a user with the `AmazonRekognitionFullAccess` permissions. `AmazonS3ReadOnlyAccess` is required for examples that access images or videos that are stored in an Amazon S3 bucket. The Amazon Rekognition Video stored video code examples also require `AmazonSQSFullAccess` permissions. 

## 3. Local CLI and SDK Installation + Setup

### Install the SDK

- From your terminal, install `boto3` (the AWS Python SDK) with `pip`.

```bash
$ pip install boto3
```

### Create an access key for the user you created in *Create an IAM User*.

- Sign in to the AWS Management Console and open the [IAM console](https://console.aws.amazon.com/iam/)

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

- In the `.aws` directory, create a new file named `credentials`.

- Open the `credentials` CSV file that you created earlier and copy its contents into the new credentials file using the following format:
 
```
[default]
aws_access_key_id = your_access_key_id
aws_secret_access_key = your_secret_access_key
```

_Substitute your acccess key ID and secret access key for your_access_key_id and your_secret_access_key._

- Save the `credentials` file and delete the older CSV file.

## 4. Application Code

Rekognition has a large number of functionalities including:

- Labels Detection
- Faces Detection
- Faces Comparison
- Faces Indexing
- Faces Search
- Text Detection


For this tutorial we'll be using the *Detect Labels* method, but if you're interested in the other functions, you can find sample code for those [available here](https://gist.github.com/alexcasalboni/0f21a1889f09760f8981b643326730ff).

### Write Python code

- The first bit of code that we'll want to write is a short script that uploads an image to `Amazon S3`. Once uploaded, we can use the uploaded object in S3 as the source for the `Rekognition` API call.

- Open up your text editor of choice and create a new python file called `pic-upload.py`.

```python3
import boto3

def create_bucket(bucket_name, region=None):
    try:
        s3_client = boto3.client('s3')
        s3_client.create_bucket(Bucket=bucket_name)

    # catch error
    except ClientError as e:
        logging.error(e)
        return False
    return True

def upload_image(file_name, bucket, object_name)
	try:
        s3_client = boto3.client('s3')
        response = s3_client.upload_file(file_name, bucket, object_name)
    except:
        return False
    return True


# This name must be GLOBALLY unique or the bucket will not create! Something like a combination of the date, your initials, and the time would work best
# ex: my_bucket_name = '01312020NW1337'
my_bucket_name = "NAME-OF-YOUR-S3-BUCKET" 
local_file_name = "name-of-file-locally.jpg"
object_name = "desired-image-name-on-upload.jpg"

create_bucket(my_bucket_name)
upload_image(local_file_name, my_bucket_name, object_name)
```

*Note:* A common error here is attempting to create a bucket with a name that is not truly globally unique.

- Save the file.

- Create a new python file called `detection-script.py`

- Paste the following code inside it.

```python3
import boto3

# make sure to copy your bucket name properly
BUCKET = "amazon-rekognition"
# submit the filename for the image in your S3 bucket?
KEY = "test.jpg"

# define a function that will accept a bucket name, a key (image name), and maximum number of labels to return
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

# call the detect_labels method, and iterate across all label results, printing each label and confidence interval
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

- Open a terminal and `cd` into the directory where your two Python scripts reside.

In your terminal:

The first script will create an S3 bucket and upload an image to S3. 

- `$ python pic-upload.py`

The second script will invoke the `Rekognition` API via the Python SDK, returning the labels for the image we uploaded.

- `$ python detection-script.py`

*Note:* For repeated use, once the bucket has been created, it's advised to comment out the `create_bucket(my_bucket_name)` method invokation at the bottom of `pic-upload.py`, as you only need to create the bucket once, and future attempts may cause errors/exceptions.


## Finished

Congrats! You just signed up for an AWS account, setup proper `IAM` credentials, created your own `Amazon S3` bucket, uploaded an image to `S3`, and executed Python code that calls the `Amazon Rekognition` Service to detect the labels within your image.

If you enjoyed this workshop, the experience for getting started with our other AI services is nearly identical! To get hands on and try them out, check out [these demoes here!](https://ai-service-demos.go-aws.com/)

### Other Resources

- [Setup for CLI and Other SDKs](https://docs.aws.amazon.com/rekognition/latest/dg/setup-awscli-sdk.html)
- [AWS Rekognition Docs](https://docs.aws.amazon.com/rekognition/index.html)
- [Other AI Services/APIs](https://aws.amazon.com/machine-learning/ai-services/)

