# ansible-role-cfn-lambda

## Role Parameters
| Variable        | Required           | Default  | Description |
| ------------- |:--------------:| -----:| -------------------------------------------------------------------------:|
| lambda_function_name | yes | | The name to use with the lambda function. |
| lambda_description | yes | | A brief description of the lambda function. |
| lambda_region | yes | | Region to deploy the lambda function into. |
| lambda_stack_name | yes | | Name of the CloudFormation stack. |
| lambda_runtime | yes | | Runtime for the lambda function. Only python runtimes are currently supported. |
| lambda_role_policies | yes | | List of policies to apply to the lambda execution role. |
| lambda_tags | yes | | List of tags to apply to the CloudFormation stack |
| lambda_create_bundle | conditional | | If the lambda function has external dependencies, <strong>yes</strong> instructs role to package them in a virtual environment. Required if <strong>lambda_source_code</strong> and <strong>lambda_source_code_file</strong> are not defined. |
| lambda_source_code | conditional | | Inline source code for lambda function. Required if <strong>lambda_source_code_file</strong> and <strong>lambda_create_bundle</strong> are not defined. |
| lambda_source_code_file | conditional | | File containing the source code to use with the lambda function. Required if <strong>lambda_source_code</strong> and <strong>lambda_create_bundle</strong> are not defined. |
| lambda_code_s3_bucket | conditional | | If deploying a packaged bundle to s3, use the this bucket. Required if creating bundle. |
| lambda_code_zipfile | conditional | | Name of the zip file to build / upload. Required if creating bundle. |
| lambda_code_s3_key | no | <strong>lambda_code_zipfile</strong> | S3 key for bundled zipfile. Used for creating a bundle. |
| lambda_build_dir | no | "build" | Name of directory to place build related files. |
| lambda_template_dir | no | <strong>lambda_build_dir</strong> | Name of directory to place rendered templates. |
| lambda_virtualenv_dir | no | "venv" | Name of virtual environment into which to build function package. |
| lambda_security_token | no | | If a security token is used for the deployment, define it here. |
| lamdda_code_s3_object_version | no | | Version of s3 object for lambda function bundle to use. |
| lambda_role_path | no | | If desired, path to use for the lambda execution role. |
| lambda_dependencies | no | | If deploying a packaged bundle to s3, list of files that need to be included in the bundle |
| lambda_requirements_file | no | | Name of pip requirements file |
| lambda_test_code | no |  | <strong>Yes specifies that a testing script should be run |
| lambda_testing_script | conditional | | A script to test the lambda function prior to deployment (i.e. python-lambda-local). If <strong>lambda_test_code</strong> is set to <strong>yes</strong> this must be defined. |
| lambda_cloudwatch_rules | no | | A list of CloudWatch Event Rules to execute the lambda. |


### lambda_role_policies Fields
| Variable        | Required           | Default  | Description |
| ------------- |:--------------:| -----:| -------------------------------------------------------------------------:|
| name | yes | | Name of the execution role policy. |
| actions | yes | | Actions granted / denied to the execution role. |
| resource | yes | | Resource to grant access to. |
| effect | yes | | Whether to <strong>Allow</strong> or <strong>Deny</strong> access to the role. |

### lambda_cloudwatch_rules Fields
| Variable        | Required           | Default  | Description |
| ------------- |:--------------:| -----:| -------------------------------------------------------------------------:|
| name | yes | | A basic name for the rule. |
| description | yes | | A description for the rule. |
| schedule | yes | | Schedule to use with the rule. |
| state | no | ENABLED | State of the rule. |


## Example Playbooks
### Source Code in File
```
- hosts: localhost
  vars:
    lambda_security_token: "{{ lookup('env','AWS_SESSION_TOKEN') }}"
    lambda_function_name: "{{ lambda_stack_name }}"
    lambda_description: "test-lambda"
    lambda_region: us-west-2
    lambda_stack_name: lambda-role-file-test
    lambda_runtime: "python2.7"
    lambda_source_code_file_contents: "{{ lookup('file', lambda_source_code_file) }}"
    lambda_role_policies:
      - name: test-policy
        actions:
          - "es:*"
        effect: "Allow"
        resource: "*"
    lambda_source_code_file: "test_function.py"
    lambda_tags:
      test: "test"
  roles:
    - lambda

```

### Inline Source Code
```
- hosts: localhost
  gather_facts: False
  vars:
    lambda_security_token: "{{ lookup('env','AWS_SESSION_TOKEN') }}"
    lambda_function_name: "{{ lambda_stack_name }}"
    lambda_description: "test-lambda"
    lambda_region: us-west-2
    lambda_stack_name: lambda-role-inline-test
    lambda_runtime: "python2.7"
    lambda_role_policies:
      - name: test-policy
        actions:
          - "es:*"
        effect: "Allow"
        resource: "*"
    lambda_source_code:
      - "def handler(event, context):"
      - " print(\"Hello World!\")"
    lambda_tags:
      test: "test"
    lambda_cloudwatch_rules:
      - name: "BasicRule"
        description: "BasicRule"
        schedule: "rate(10 minutes)"
        state: "ENABLED"
  roles:
    - lambda
```

### S3 Bundle
```
- hosts: localhost
  gather_facts: False
  vars:
    lambda_create_bundle: "yes"
    lambda_virtualenv_dir: "testing"
    lambda_security_token: "{{ lookup('env','AWS_SESSION_TOKEN') }}"
    lambda_function_name: "{{ lambda_stack_name }}"
    lambda_description: "test-lambda-bundle"
    lambda_code_s3_bucket: "role-testing-bucket"
    lambda_code_zipfile: "test.zip"
    lambda_code_s3_object_version: "QDScRvyhIEdFM0i.AZLF8OANt.U6_vGy"
    lambda_region: us-west-2
    lambda_role_path: "/test/"
    lambda_requirements:
      - "{{ lambda_requirements_file }}"
    lambda_stack_name: lambda-role-bundle-test
    lambda_runtime: "python2.7"
    lambda_requirements_file: "test_requirements.pip"
    lambda_role_policies:
      - name: test-policy
        actions:
          - "es:*"
        effect: "Allow"
        resource: "*"
    lambda_tags:
      test: "test"
  roles:
    - lambda
```
