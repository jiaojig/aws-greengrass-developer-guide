# Setting Configuration to Run the AWS IoT Greengrass Qualification Suite<a name="set-config"></a>

Before you run tests, you must configure settings for AWS credentials and devices on your host computer\.

## Configure Your AWS Credentials<a name="cfg-aws-gg"></a>

You must configure your IAM user credentials in the `<device_tester_extract_location> /configs/config.json` file\. Use the credentials for the IDT for AWS IoT Greengrass specific user created in [Create and Configure an AWS Account](dev-tst-prereqs.md#config-aws-account)\. You can specify your credentials in one of two ways:
+ Credentials file
+ Environment variables

### Configure AWS Credentials with a Credentials File<a name="config-cred-file"></a>

IDT uses the same credentials file as the AWS CLI\. For more information, see [Configuration and Credential Files](https://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html)\.

The location of the credential file varies based on the operating system you are using:
+ macOS, Linux: `~/.aws/credentials`
+ Windows: `C:\Users\UserName\.aws\credentials`

Add your AWS credentials to the `credentials` file in the following format:

```
[default]
aws_access_key_id = <your_access_key_id>
aws_secret_access_key = <your_secret_access_key>
```

To configure IDT for AWS IoT Greengrass to use AWS credentials from your `credentials` file, edit your `config.json ` file as follows:

```
{
	"awsRegion": "us-west-2",
	"auth": {
		"method": "file",
		"credentials": {
			"profile": "default"
		}
	}
}
```

**Note**  
If you do not use the `default` AWS profile be sure to change the profile name in your config\.json file\. For more information, see [Named Profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)\.

### Configure AWS Credentials with Environment Variables<a name="config-env-vars"></a>

Environment variables are variables maintained by the operating system and used by system commands\. They will not be saved if you close the SSH session\. IDT for AWS IoT Greengrass can use the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables to store your AWS credentials\.

To set these variables on Linux, macOS, or Unix, use export:

```
export AWS_ACCESS_KEY_ID=<your_access_key_id>
export AWS_SECRET_ACCESS_KEY=<your_secret_access_key>
```

To set these variables on Windows, use set:

```
set AWS_ACCESS_KEY_ID=<your_access_key_id>
set AWS_SECRET_ACCESS_KEY=<your_secret_access_key>
```

To configure IDT to use the environment variables, edit the `auth` section in your config\.json file\. Below is an example config\.json file:

```
{
	"awsRegion": "us-west-2",
	"auth": {
		"method": "environment"
	}
}
```

## Device Configuration<a name="device-config"></a>

In addition to AWS credentials, IDT for AWS IoT Greengrass needs information about the devices that tests are run on \(for example, IP address, login information, operating system, and CPU architecture\)\.

You must provide this information using the `device.json` template located in ` <device_tester_extract_location>/configs/device.json`:

```
[
  {
    "id": "<pool-id>",
    "sku": "<sku>",
    "features": [
      {
        "name": "os",
        "value": "linux | ubuntu"
      },
      {
        "name": "arch",
        "value": "x86_64 | armv7l | aarch64"
      }
    ],
    "hsm": {
      "p11Provider": "</path/to/pkcs11ProviderLibrary>",
      "slotLabel": "<slot-label>",
      "slotUserPin": "<pin>",
      "privateKeyLabel": "<key-label>",
      "openSSLEngine": "</path/to/openssl/engine>"
    },
    "kernelConfigLocation": "",
    "greengrassLocation": "",
    "devices": [
      {
        "id": "<device-id>",
        "connectivity": {
          "protocol": "ssh",
          "ip": "<ip-address>",
          "auth": {
            "method": "pki",
            "credentials": {
              "user": "<user>",
              "privKeyPath": "</path/to/private/key>"
            }
          }
        }
      }
    ]
  }
]
```

All fields that contain values are required as described here:

`id`  
A user\-defined alphanumeric ID that uniquely identifies a collection of devices called a device pool\. Devices that belong to a pool must have identical hardware\. When you run a suite of tests, devices in the pool are used to parallelize the workload, multiple devices are used to run different tests\.

`sku`  
An alphanumeric value that uniquely identifies the device under test\. The SKU is used to track qualified boards\.  
If you want to list your board in the AWS Partner Device Catalog, the SKU you specify here must match the SKU that you use in the listing process\.

`features`  
An array that contains the device's supported features\.  
+ Required features: `os`, `arch`\.
+ Supported OS/architecture combinations:
  + Linux, x86\_64
  + Linux, ARMv7l
  + Linux, AArch64
  + Ubuntu, x86\_64

`hsm (optional)`  
Contains configuration information for testing with a AWS IoT Greengrass Hardware Security Module \(HSM\), otherwise the `<hsm>` element should be omitted\. For more information, see [Hardware Security Integration](hardware-security.md)\.    
`hsm.p11Provider`  
The absolute path to the PKCS\#11 implementation's libdl\-loadable library\.  
`hsm.slotLabel`  
The slot label used to identify the hardware module\.  
`hsm.slotUserPin`  
The user pin used to authenticate the AWS IoT Greengrass core to the module\.  
`hsm.privateKeyLabel`  
The label used to identify the key in the hardware module\.  
`hsm.openSSLEngine`  
The absolute path to the OpenSSL engine's \.so file that enables PKCS\#11 support on OpenSSL\. This is used by the AWS IoT Greengrass OTA update agent\.

`devices.id`  
A user\-defined unique identifier for the device being tested\.

`connectivity.protocol`  
The communication protocol used to communicate with this device\. Currently the only supported value is `ssh`

`connectivity.ip`  
The IP address of the device being tested\.

`connectivity.auth.method`  
The authorization method used to access a device over the given connectivity protocol\. Currently the only supported value is `pki`\.

`connectivity.auth.credentials.user`  
The user name for signing in to the device being tested\.

`connectivity.auth.credentials.privKeyPath`  
The full path to the private key used to sign in to the device being tested\.

`greengrassLocation`  
This value is only used when you use an existing installation of AWS IoT Greengrass\. The location of AWS IoT Greengrass Core software on your devices\. Use this attribute to tell IDT to use the version of the AWS IoT Greengrass Core software installed on your devices\.

`kernelConfigLocation`  
\(Optional\) The path to the kernel configuration file\. AWS IoT Device Tester uses this file to check if the devices have the required kernel features enabled\. If not specified, IDT uses the following paths: `/proc/config.gz` and `/boot/config-<kernel-version>` to search for the kernel configuration file\. It uses the first one it finds\.