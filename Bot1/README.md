# twitterPollerRetweeter

You will find here the main files for the application:
hello_World/
  * twitterRetweet.py -- main file for code source
  * requirements.txt -- the list of the library needed

**NOTE**: It is recommended to use a Python Virtual environment to separate your application development from  your system Python installation.

**Bot 1: Make a bot who retweet automatically the tweets based on keywords**
1. Gather the twitter credentials, generated in the pre requisites
2. Look at the lambda code available in this repository.
3. Create a KMS key (used to encrypt twitter credentials)
4. Create 4 SSM parameters with twitter credentials and use the KMS key
5. Create an S3 bucket to host your code package
6. Create the lambda package
    1. Use SAM and init a project
```sam init --runtime python3.6```
    2. Update template.yaml
    3. Copy/paste the code in app.py
    4. Update app.py with the name of the SSM variables
    5. Update requirements.txt with needed libraries (sample provided in this repository)
    6. Package code, first get the packages updated
    7. ```pip install -r requirements.txt -t hello_world/build/```
    8. ```cp hello_world/*.py hello_world/build/```
    
7. Package the Lambda and put it in your s3 bucket for deployment:
```sam package --template-file template.yaml --output-template-file packaged.yaml --s3-bucket REPLACE_THIS_WITH_YOUR_S3_BUCKET_NAME```
8. Deploy the lambda with a proper stack name and enable IAM capability to create roles automatically
```sam deploy --template-file packaged.yaml --stack-name twitterpollerretweeter --capabilities CAPABILITY_IAM```
9. Let's update the rights in IAM role created for the Lambda and allow access to KMS, SSM parameters
10. Deploy with Alias in template.yaml in order to update the code.
11. If you do some code update, copy code only in build folder and redo sam package and sam deploy

**We could instrument a bit this function to leverage X-Ray and understand where we are spending the time:**

12. Enable tracing with X-ray. Could do in Lambda console or sam file.
13. Import X-ray sdk in your project
14. Patch_all() will help to patch all boto3 and managed libraries
13. Enable some sub-segment in the code if you want more details.

Sample of instrumented function in the repository.


# Appendix

### Python Virtual environment
**In case you're new to this**, python3 comes with `virtualenv` library by default so you can simply run the following:

1. Create a new virtual environment
2. Install dependencies in the new virtual environment

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
```


**NOTE:** You can find more information about Virtual Environment at [Python Official Docs here](https://docs.python.org/3/tutorial/venv.html). Alternatively, you may want to look at [Pipenv](https://github.com/pypa/pipenv) as the new way of setting up development workflows
## AWS CLI commands

AWS CLI commands to package, deploy and describe outputs defined within the cloudformation stack:

```bash
sam package \
    --template-file template.yaml \
    --output-template-file packaged.yaml \
    --s3-bucket REPLACE_THIS_WITH_YOUR_S3_BUCKET_NAME

sam deploy \
    --template-file packaged.yaml \
    --stack-name twitterpollerretweeter \
    --capabilities CAPABILITY_IAM \
    --parameter-overrides MyParameterSample=MySampleValue

aws cloudformation describe-stacks \
    --stack-name twitterpollerretweeter --query 'Stacks[].Outputs'
```