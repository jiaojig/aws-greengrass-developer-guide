# Run Lambda Functions on the AWS IoT Greengrass Core<a name="lambda-functions"></a>

AWS IoT Greengrass provides a containerized Lambda runtime environment for user\-defined code\. Lambda functions that are deployed to an AWS IoT Greengrass core run in the core's local Lambda runtime\. Local Lambda functions can be triggered by local events, messages from the cloud, and other sources, which brings local compute functionality to connected devices\. For example, you can use Greengrass Lambda functions to filter device data before transmitting the data to the cloud\.

To deploy a Lambda function to a core, you add the function to a Greengrass group \(by referencing the existing Lambda function\), configure group\-specific settings for the function, and then deploy the group\. If the function accesses AWS services, you also must add any required permissions to the group role\.

You can configure parameters that determine how the Lambda functions run, including permissions, isolation, memory limits, and more\. For more information, see [Controlling Execution of Greengrass Lambda Functions by Using Group\-Specific Configuration](lambda-group-config.md)\. These settings can also enable you to run AWS IoT Greengrass in a Docker container\. For more information, see [Running AWS IoT Greengrass in a Docker Container](run-gg-in-docker-container.md)\.

## SDKs for Greengrass Lambda Functions<a name="lambda-sdks"></a>

AWS provides three SDKs that can be used by Greengrass Lambda functions running on an AWS IoT Greengrass core\. These SDKs are contained in different packages, so functions can use them simultaneously\. To use an SDK in a Greengrass Lambda function, you must include it in your deployment package\.

**AWS IoT Greengrass Core SDK**  <a name="lambda-sdks-core"></a>
The AWS IoT Greengrass Core SDK enables Lambda functions to interact with the core device, publish messages to AWS IoT, interact with the local shadow service, invoke other deployed Lambda functions, and access secret resources\.  
The following table lists supported languages or platforms and the versions of AWS IoT Greengrass core software that it can run on\.    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lambda-functions.html)
The following table lists the changes introduced in AWS IoT Greengrass Core SDK versions for Java, Node\.js, and Python\.    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lambda-functions.html)
For download information, see [AWS IoT Greengrass Core SDK Software](what-is-gg.md#gg-core-sdk-download)\.  
If you're running Python Lambda functions, you can use [https://pypi.org/project/pip/](https://pypi.org/project/pip/) to install the AWS IoT Greengrass Core SDK for Python on the core device\. Then you can deploy your functions without including the SDK in the Lambda function deployment package\. For more information, see [greengrasssdk](https://pypi.org/project/greengrasssdk/)\.  
To use `pip` to install the Python SDK, run the following command in your core device terminal\.  

```
pip install greengrasssdk
```

**AWS IoT Greengrass Machine Learning SDK**  <a name="lambda-sdks-ml"></a>
Current version: 1\.0\.0  
The AWS IoT Greengrass Machine Learning SDK enables Lambda functions to consume machine learning models that are deployed to the Greengrass core as machine learning resources\. Lambda functions can use the SDK to invoke and interact with a local inference service that's deployed to the core as a connector\. For more information, including a code example that uses the SDK, see the [Image Classification](image-classification-connector.md) connector\.  
The following table lists supported languages or platforms and the versions of AWS IoT Greengrass core software that it can run on\.    
****    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/greengrass/latest/developerguide/lambda-functions.html)
For download information, see [AWS IoT Greengrass ML SDK Software](what-is-gg.md#gg-ml-sdk-download)\.

**AWS SDKs**  <a name="lambda-sdks-aws"></a>
Enables local Lambda functions to make direct calls to AWS services, such as Amazon S3, DynamoDB, AWS IoT, and AWS IoT Greengrass\. To use an AWS SDK in a Greengrass Lambda function, you must include it in your deployment package\. When you use the AWS SDK in the same package as the AWS IoT Greengrass Core SDK, make sure that your Lambda functions use the correct namespaces\. Greengrass Lambda functions can't communicate with cloud services when the core is offline\.  
Download the appropriate AWS SDKs from the [ Getting Started Resource Center](https://aws.amazon.com/getting-started/tools-sdks/)\.

For more information about creating a deployment package, see [Create and Package a Lambda Function](create-lambda.md) in the Getting Started tutorial or [Creating a Deployment Package](https://docs.aws.amazon.com/lambda/latest/dg/deployment-package-v2.html) in the *AWS Lambda Developer Guide*\.

### Migrating Cloud\-Based Lambda Functions<a name="lambda-migrate-sdks"></a>

The AWS IoT Greengrass Core SDK follows the AWS SDK programming model, which makes it easy to port Lambda functions that are developed for the cloud to Lambda functions that run on an AWS IoT Greengrass core\.

For example, the following Python Lambda function uses the AWS SDK for Python to publish a message to the topic *some/topic* in the cloud:

```
import boto3
        
client = boto3.client('iot-data')
response = client.publish(
    topic = 'some/topic',
    qos = 0,
    payload = 'Some payload'.encode()
)
```

To port the function for an AWS IoT Greengrass core, in the `import` statement and `client` initialization, change the `boto3` module name to `greengrasssdk`, as shown in the following example:

```
import greengrasssdk
                        
client = greengrasssdk.client('iot-data')
response = client.publish(
    topic = 'some/topic',
    qos = 0,
    payload = 'Some payload'.encode()
)
```

**Note**  
The AWS IoT Greengrass Core SDK supports sending MQTT messages with QoS = 0 only\.

The similarity between programming models also makes it possible for you to develop your Lambda functions in the cloud and then migrate them to AWS IoT Greengrass with minimal effort\. [Lambda executables](#lambda-executables) don't run in the cloud, so you can't use the AWS SDK to develop them in the cloud before deployment\.

## Reference Lambda Functions by Alias or Version<a name="lambda-versions-aliases"></a>

Greengrass groups can reference a Lambda function by alias \(recommended\) or by version\. Using an alias makes it easier to manage code updates because you don't have to change your subscription table or group definition when the function code is updated\. Instead, you just point the alias to the new function version\. Aliases resolve to version numbers during group deployment\. When you use aliases, the resolved version is updated to the version that the alias is pointing to at the time of deployment\.

AWS IoT Greengrass doesn't support Lambda aliases for **$LATEST** versions\. **$LATEST** versions aren't bound to immutable, published function versions and can be changed at any time, which is counter to the AWS IoT Greengrass principle of version immutability\.

A common practice for keeping your Greengrass Lambda functions updated with code changes is to use an alias named **PRODUCTION** in your Greengrass group and subscriptions\. As you promote new versions of your Lambda function into production, point the alias to the latest stable version and then redeploy the group\. You can also use this method to roll back to a previous version\.

## Communication Flows for Greengrass Lambda Functions<a name="lambda-communication"></a>

Greengrass Lambda functions support several methods of communicating with other members of the AWS IoT Greengrass group, local services, and cloud services \(including AWS services\)\.

### Communication Using MQTT Messages<a name="lambda-messages"></a>

Lambda functions can send and receive MQTT messages using a publish\-subscribe pattern that's controlled by subscriptions\.

This communication flow allows Lambda functions to exchange messages with the following entities:
+ Devices in the group\.
+ Connectors in the group\.
+ Other Lambda functions in the group\.
+ AWS IoT\.
+ Local Device Shadow service\.

A subscription defines a message source, a message target, and a topic \(or subject\) that's used to route messages from the source to the target\. Messages that are published to a Lambda function are passed to the function's registered handler\. Subscriptions enable more security and provide predictable interactions\. For more information, see [Greengrass Messaging Workflow](gg-sec.md#gg-msg-workflow)\.

**Note**  
When the core is offline, Greengrass Lambda functions can exchange messages with devices, connectors, other functions, and local shadows, but messages to AWS IoT are queued\. For more information, see [MQTT Message Queue](gg-core.md#mqtt-message-queue)\.

### Other Communication Flows<a name="lambda-other-communication"></a>
+ To interact with local resources and machine learning models on a core device, Greengrass Lambda functions use platform\-specific operating system interfaces\. For example, you can use the `open` method in the [os](https://docs.python.org/2/library/os.html) module in Python 2\.7 functions\. To allow a function to access a resource, the function must be *affiliated* with the resource and granted `read-only` or `read-write` permission\. For more information, including AWS IoT Greengrass core version availability, see [Access Local Resources with Lambda Functions](access-local-resources.md) and [Perform Machine Learning Inference](ml-inference.md)\.
**Note**  
If you run your Lambda function without containerization, you cannot use attached local resources and must access those resources directly\. Machine learning resources are not available for Lambda functions that run without containerization\.
+ Greengrass Lambda functions can use the AWS SDK to communicate with AWS services\. For more information, see [AWS SDK](#aws-sdk)\.
+ Greengrass Lambda functions can use third\-party interfaces to communicate with external cloud services, similar to cloud\-based Lambda functions\.

**Note**  
Greengrass Lambda functions can't communicate with AWS or other cloud services when the core is offline\.

## Retrieve the Input MQTT Topic \(or Subject\)<a name="lambda-get-mqtt-topic"></a>

AWS IoT Greengrass uses subscriptions to control the exchange of MQTT messages between devices, Lambda functions, and connectors in a group, and with AWS IoT or the local shadow service\. Subscriptions define a message source, message target, and an MQTT topic used to route messages\. When the target is a Lambda function, the function's handler is invoked when the source publishes a message\. For more information, see [Communication Using MQTT Messages](#lambda-messages)\.

The following example shows how a Lambda function can get the input topic from the `context` that's passed to the handler\. It does this by accessing the `subject` key from the context hierarchy \(`context.client_context.custom['subject']`\)\. The example also parses the input JSON message and then publishes the parsed topic and message\.

**Note**  
In the AWS IoT Greengrass API, the topic of a [subscription](https://docs.aws.amazon.com/greengrass/latest/apireference/definitions-subscription.html) is represented by the `subject` property\.

```
import greengrasssdk
import logging

client = greengrasssdk.client('iot-data')

OUTPUT_TOPIC = 'test/topic_results'

def get_input_topic(context):
    try:
        topic = context.client_context.custom['subject']
    except Exception as e:
        logging.error('Topic could not be parsed. ' + repr(e))
    return topic
    
def get_input_message(event):
    try:
        message = event['test-key']
    except Exception as e:
        logging.error('Message could not be parsed. ' + repr(e))
    return message

def function_handler(event, context):
    try:
        input_topic = get_input_topic(context)
        input_message = get_input_message(event)
        response = 'Invoked on topic "%s" with message "%s"' % (input_topic, input_message)
        logging.info(response)
    except Exception as e:
        logging.error(e)

    client.publish(topic=OUTPUT_TOPIC, payload=response)

    return
```

To test the function, add it to your group using the default configuration settings\. Then, add the following subscriptions and deploy the group\. For instructions, see [Module 3 \(Part 1\): Lambda Functions on AWS IoT Greengrass](module3-I.md)\.


****  

| Source | Target | Topic filter | 
| --- | --- | --- | 
| IoT Cloud | This function | test/input\_message | 
| This function | IoT Cloud | test/topic\_results | 

After the deployment is completed, invoke the function\.

1. In the AWS IoT Core console, open the **Test** page\.

1. Subscribe to the `test/topic_results` topic\.

1. Publish a message to the `test/input_message` topic\. For this example, you must include the `test-key` property in the JSON messsage\.

   ```
   {
     "test-key": "Some string value"
   }
   ```

   If successful, the function publishes the input topic and message string to the `test/topic_results` topic\.

## Lifecycle Configuration for Greengrass Lambda Functions<a name="lambda-lifecycle"></a>

The Greengrass Lambda function lifecycle determines when a function starts and how it creates and uses containers\. The lifecycle also determines how variables and preprocessing logic that are outside of the function handler are retained\.

AWS IoT Greengrass supports the on\-demand \(default\) or long\-lived lifecycles:
+ **On\-demand** functions start when they are invoked and stop when there are no tasks left to execute\. An invocation of the function creates a separate container \(or sandbox\) to process invocations, unless an existing container is available for reuse\. Data that's sent to the function might be pulled by any of the containers\.

  Multiple invocations of an on\-demand function can run in parallel\.

  Variables and preprocessing logic that are defined outside of the function handler are not retained when new containers are created\.
+ **Long\-lived** \(or *pinned*\) functions start automatically when the AWS IoT Greengrass core starts and run in a single container\. All data that's sent to the function is pulled by the same container\.

  Multiple invocations are queued until earlier invocations are executed\.

  Variables and preprocessing logic that are defined outside of the function handler are retained for every invocation of the handler\.

  Long\-lived Lambda functions are useful when you need to start doing work without any initial input\. For example, a long\-lived function can load and start processing an ML model to be ready when the function starts receiving device data\.
**Note**  
Remember that long\-lived functions have timeouts that are associated with invocations of their handler\. If you want to execute indefinitely running code, you must start it outside the handler\. Make sure that there's no blocking code outside the handler that might prevent the function from completing its initialization\.  
 These functions run unless the core stops \(for example during a group deployment or a device reboot\) or the function enters an error state \(such as a handler timeout, uncaught exception, or when it exceeds its memory limits\)\.

For more information about container reuse, see [Understanding Container Reuse in AWS Lambda](https://aws.amazon.com/blogs/compute/container-reuse-in-lambda/) on the AWS Compute Blog\.

## Lambda Executables<a name="lambda-executables"></a>

This feature is available for AWS IoT Greengrass Core v1\.6 and later\.

A Lambda executable is a type of Greengrass Lambda function that you can use to run binary code in the core environment\. It lets you execute device\-specific functionality natively, and benefit from the smaller footprint of compiled code\. Lambda executables can be invoked by events, invoke other functions, and access local resources\.

Lambda executables support the binary encoding type only \(not JSON\), but otherwise you can manage them in your Greengrass group and deploy them like other Greengrass Lambda functions\. However, the process of creating Lambda executables is different from creating Python, Java, and Node\.js Lambda functions:
+ You can't use the AWS Lambda console to create \(or manage\) a Lambda executable\. You can create a Lambda executable only by using the AWS Lambda API\.
+ You upload the function code to AWS Lambda as a compiled executable that includes the [AWS IoT Greengrass Core SDK for C](https://github.com/aws/aws-greengrass-core-sdk-c)\.
+ You specify the executable name as the function handler\.

Lambda executables must implement certain calls and programming patterns in their function code\. For example, the `main` method must:
+ Call `gg_global_init` to initialize Greengrass internal global variables\. This function must be called before creating any threads, and before calling any other AWS IoT Greengrass Core SDK functions\.
+ Call `gg_runtime_start` to register the function handler with the Greengrass Lambda runtime\. This function must be called during initialization\. Calling this function causes the current thread to be used by the runtime\. The optional `GG_RT_OPT_ASYNC` parameter tells this function to not block, but instead to create a new thread for the runtime\. This function uses a `SIGTERM` handler\.

The following snippet is the `main` method from the [simple\_handler\.c](https://github.com/aws/aws-greengrass-core-sdk-c/blob/master/aws-greengrass-core-sdk-c-example/simple_handler.c) code example on GitHub\.

```
int main() {
    gg_error err = GGE_SUCCESS;

    err = gg_global_init(0);
    if(err) {
        gg_log(GG_LOG_ERROR, "gg_global_init failed %d", err);
        goto cleanup;
    }

    gg_runtime_start(handler, 0);

cleanup:
    return -1;
}
```

For more information about requirements, constraints, and other implementation details, see [AWS IoT Greengrass Core SDK for C](https://github.com/aws/aws-greengrass-core-sdk-c)\.

### Create a Lambda Executable<a name="create-lambda-executable"></a>

After you compile your code along with the SDK, use the AWS Lambda API to create a Lambda function and upload your compiled executable\.

**Note**  
Your function must be compiled with a C89 compatible compiler\.

The following example uses the [create\-function](https://docs.aws.amazon.com/cli/latest/reference/lambda/create-function.html) CLI command to create a Lambda executable\. The command specifies:
+ The name of the executable for the handler\. This must be the exact name of your compiled executable\.
+ The path to the `.zip` file that contains the compiled executable\.
+ `arn:aws:greengrass:::runtime/function/executable` for the runtime\. This is the runtime for all Lambda executables\.

**Note**  
For `role`, you can specify the ARN of any Lambda execution role\. AWS IoT Greengrass doesn't use this role, but the parameter is required to create the function\. For more information about Lambda execution roles, see [AWS Lambda Permissions Model](https://docs.aws.amazon.com/lambda/latest/dg/intro-permission-model.html) in the *AWS Lambda Developer Guide*\.

```
aws lambda create-function \
--region aws-region \
--function-name function-name \
--handler executable-name \
--role role-arn \
--zip-file fileb://file-name.zip \
--runtime arn:aws:greengrass:::runtime/function/executable
```

Next, use the AWS Lambda API to publish a version and create an alias\.
+ Use [publish\-version](https://docs.aws.amazon.com/cli/latest/reference/lambda/publish-version.html) to publish a function version\.

  ```
  aws lambda publish-version \
  --function-name function-name \
  --region aws-region
  ```
+ Use [create\-alias](https://docs.aws.amazon.com/cli/latest/reference/lambda/create-alias.html) to create an alias the points to the version you just published\. We recommend that you reference Lambda functions by alias when you add them to a Greengrass group\.

  ```
  aws lambda create-alias \
  --function-name function-name \
  --name alias-name \
  --function-version version-number \
  --region aws-region
  ```

**Note**  
The AWS Lambda console doesn't display Lambda executables\. To update the function code, you must also use the Lambda API\.

Then, add the Lambda executable to a Greengrass group, configure it to accept binary input data in its group\-specific settings, and deploy the group\. You can do this in the AWS IoT Greengrass console or by using the AWS IoT Greengrass API\.