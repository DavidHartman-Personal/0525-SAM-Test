# AWS Severless Lambda Function Template

This project contains source code and supporting files for a serverless application that you can deploy with the SAM CLI. It includes the following files and folders.

- hello_world - Code for the application's Lambda function.
- events - Invocation events that you can use to invoke the function.
- tests - Unit tests for the application code. 
- template.yaml - A template that defines the application's AWS resources.

The application uses several AWS resources, including Lambda functions and an API Gateway API. These resources are defined in the `template.yaml` file in this project. You can update the template to add AWS resources through the same deployment process that updates your application code.

If you prefer to use an integrated development environment (IDE) to build and test your application, you can use the AWS Toolkit.  
The AWS Toolkit is an open source plug-in for popular IDEs that uses the SAM CLI to build and deploy serverless applications on AWS. The AWS Toolkit also adds a simplified step-through debugging experience for Lambda function code. See the following links to get started.

* [IntelliJ](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [PyCharm](https://docs.aws.amazon.com/toolkit-for-jetbrains/latest/userguide/welcome.html)
* [VS Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/welcome.html)

## Deploy the sample application

The Serverless Application Model Command Line Interface (SAM CLI) is an extension of the AWS CLI that adds functionality for building and testing Lambda applications. It uses Docker to run your functions in an Amazon Linux environment that matches Lambda. It can also emulate your application's build environment and API.

Prior to running the commands noted below, define/export some environment variables.  This assumes that WSL/Unix is 
being used.

```bash
root_dir="/mnt/c/Users/david/PycharmProjects/0525-SAM-Test/"
AWS_PROFILE="dave-personal"
date_time=$(date +'%Y_%m_%d_%H_%M_%S')
logs_dir=${root_dir}/logs
output_dir=${root_dir}/output
log_suffix="0525-SAM-Test"  # this is a default log file suffix.  Change as needed.
log_file="${logs_dir}/${log_suffix}_${date_time}.log"
# sam variables
stack_name="sam-app-hello-world-test-05252022"
stack_logical_resource_id="HelloWorldFunction"
```

HOLD OFF ON THIS FOR NOW
The SAM config file that can be used instead of using the --guided option can be created as 

```bash
version = 0.1
[default]
[default.deploy]
[default.deploy.parameters]
stack_name = "sam-app-hello-world-test-05252022"
s3_bucket = "aws-sam-cli-managed-default-samclisourcebucket-wpu24m6n1w3q"
s3_prefix = "sam-app-hello-world-test-05252022"
region = "us-east-1"
capabilities = "CAPABILITY_IAM"
```
HOLD OFF ON THIS FOR NOW END

Note that the sam build can be run without the container if the installed venv/ is the same vesrion as needed for 
the functions.

To build and deploy your application for the first time, run the following in your shell

```bash
sam build --use-container
sam deploy --guided

# or
sam build
sam deploy
```

The first command will build the source of your application. By using --use-container, a Docker image is downloaded 
and started to create the deployment package.

The second command will package and deploy your application to AWS, with a series of prompts:

* **Stack Name**: The name of the stack to deploy to CloudFormation. This should be unique to your account and region, and a good starting point would be something matching your project name.
* **AWS Region**: The AWS region you want to deploy your app to.
* **Confirm changes before deploy**: If set to yes, any change sets will be shown to you before execution for manual review. If set to no, the AWS SAM CLI will automatically deploy application changes.
* **Allow SAM CLI IAM role creation**: Many AWS SAM templates, including this example, create AWS IAM roles required for the AWS Lambda function(s) included to access AWS services. By default, these are scoped down to minimum required permissions. To deploy an AWS CloudFormation stack which creates or modifies IAM roles, the `CAPABILITY_IAM` value for `capabilities` must be provided. If permission isn't provided through this prompt, to deploy this example you must explicitly pass `--capabilities CAPABILITY_IAM` to the `sam deploy` command.
* **Save arguments to samconfig.toml**: If set to yes, your choices will be saved to a configuration file inside the project, so that in the future you can just re-run `sam deploy` without parameters to deploy changes to your application.

The outputs from this example include the Lambda Function ARN.

To get the Lambda Function resource ID, run the following:

```bash
physical_resource_id=$(aws cloudformation describe-stack-resource \
                      --stack-name=${stack_name} \
                      --logical-resource-id=${stack_logical_resource_id} \
                      --query "StackResourceDetail.PhysicalResourceId" \
                      --output text)
echo "$physical_resourc_id"                    
```

To test the newly deployed Lambda Function run the following:

```bash
log_suffix="aws-lambda-invoke-test"
log_file="${logs_dir}/${log_suffix}_$(date +'%Y_%m_%d_%H_%M_%S').log"
aws lambda invoke --function-name ${physical_resource_id} \
                  --log-type Tail \
                  --cli-binary-format raw-in-base64-out \
                  --payload file://events/event.json \
                  --output text \
                  --query "LogResult" \
                  ${log_file} |  base64 -d
echo "*** Results: ${log_file} ****"
cat ${log_file}
echo "*** End of Results ****"
```
## Use the SAM CLI to build and test locally

Build your application with the `sam build --use-container` command.

```bash
0525-SAM-Test$ sam build --use-container
```

The SAM CLI installs dependencies defined in `hello_world/requirements.txt`, creates a deployment package, and saves it in the `.aws-sam/build` folder.

Test a single function by invoking it directly with a test event. An event is a JSON document that represents the input that the function receives from the event source. Test events are included in the `events` folder in this project.

Run functions locally and invoke them with the `sam local invoke` command.

```bash
0525-SAM-Test$ sam local invoke HelloWorldFunction --event events/event.json
```

The SAM CLI can also emulate your application's API. Use the `sam local start-api` to run the API locally on port 3000.

```bash
0525-SAM-Test$ sam local start-api
0525-SAM-Test$ curl http://localhost:3000/
```

The SAM CLI reads the application template to determine the API's routes and the functions that they invoke. The `Events` property on each function's definition includes the route and method for each path.

```yaml
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

## Add a resource to your application
The application template uses AWS Serverless Application Model (AWS SAM) to define application resources. AWS SAM is an extension of AWS CloudFormation with a simpler syntax for configuring common serverless application resources such as functions, triggers, and APIs. For resources not included in [the SAM specification](https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md), you can use standard [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) resource types.

## Fetch, tail, and filter Lambda function logs

To simplify troubleshooting, SAM CLI has a command called `sam logs`. `sam logs` lets you fetch logs generated by your deployed Lambda function from the command line. In addition to printing the logs on the terminal, this command has several nifty features to help you quickly find the bug.

`NOTE`: This command works for all AWS Lambda functions; not just the ones you deploy using SAM.

```bash
0525-SAM-Test$ sam logs -n HelloWorldFunction --stack-name 0525-SAM-Test --tail
```

You can find more information and examples about filtering Lambda function logs in the [SAM CLI Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html).

## Tests

Tests are defined in the `tests` folder in this project. Use PIP to install the test dependencies and run tests.

```bash
0525-SAM-Test$ pip install -r tests/requirements.txt --user
# unit test
0525-SAM-Test$ python -m pytest tests/unit -v
# integration test, requiring deploying the stack first.
# Create the env variable AWS_SAM_STACK_NAME with the name of the stack we are testing
0525-SAM-Test$ AWS_SAM_STACK_NAME=<stack-name> python -m pytest tests/integration -v
```

## Cleanup

To delete the sample application that you created, use the AWS CLI. Assuming you used your project name for the stack name, you can run the following:

```bash
aws cloudformation delete-stack --stack-name 0525-SAM-Test
```

## Resources

See the [AWS SAM developer guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) for an introduction to SAM specification, the SAM CLI, and serverless application concepts.

Next, you can use AWS Serverless Application Repository to deploy ready to use Apps that go beyond hello world samples and learn how authors developed their applications: [AWS Serverless Application Repository main page](https://aws.amazon.com/serverless/serverlessrepo/)
