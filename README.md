# OIDC Identity Provider CloudFormation Template Custom Resource

This template and associated Lambda function and custom resource add support to \
AWS CloudFormation for the [AWS IAM OpenId Connect Identity Provider][1]
resource type.

This is copied from [Mozilla security teams's repo](https://github.com/mozilla/security/tree/master/operations/cloudformation-templates/oidc_identity_provider)
with a few modifications:
* Placed in SAM directory structure
* Modified the custom resource template with SAM properties
* Built and packaged with the SAM CLI
* Updated README.md file
* Removed Makefile and deploy.sh

## Usage

Launch the CloudFormation stack using the [sceptre](https://github.com/Sceptre/sceptre).
The sceptre template should be something like this..

```yaml
template_path: remote/cfn-oidc-identity-provider.yaml
stack_name: my-oidc-idp
stack_tags:
  Department: "Platform"
  Project: "Infrastructure"
  OwnerEmail: "joe.smith@sagebase.org"
parameters:
  Url: "https://prod.acme.org/auth/v1"
  ClientIDList: "Client1"
  ThumbprintList: "09aa48a9a6fb14926bb7f3fa2e02da2b0ab02fa"
hooks:
  before_launch:
    - !cmd "curl https://s3.amazonaws.com/bootstrap-awss3cloudformationbucket-19qromfd235z9/aws-infra/master/cfn-oidc-identity-provider.yaml --create-dirs -o templates/remote/cfn-oidc-identity-provider.yaml"
```

This will launch the stack with example URL, Client IDs and Thumbprints.

## Why a separate Lambda file

The code to handle creation updating and deletion of the OIDC Identity Provider
couldn't fit into the [4096 characters][2] allowed for embedded code without
heavily obfuscating the code.


## Development

### Contributions
Contributions are welcome.

### Requirements
Run `pipenv install --dev` to install both production and development
requirements, and `pipenv shell` to activate the virtual environment. For more
information see the [pipenv docs](https://pipenv.pypa.io/en/latest/).

After activating the virtual environment, run `pre-commit install` to install
the [pre-commit](https://pre-commit.com/) git hook.

### Create a local build

```shell script
$ sam build --use-container
```

### Run locally

```shell script
$ sam local invoke IdentityProviderCreatorFunction --event events/event.json
```

### Run unit tests
Tests are defined in the `tests` folder in this project. Use PIP to install the
[pytest](https://docs.pytest.org/en/latest/) and run unit tests.

```shell script
$ python -m pytest tests/ -v
```

## Deployment

### Build

```shell script
sam build
```

## Deploy Lambda to S3
This requires the correct permissions to upload to bucket
`bootstrap-awss3cloudformationbucket-19qromfd235z9` and
`essentials-awss3lambdaartifactsbucket-x29ftznj6pqw`

```shell script
sam package --template-file .aws-sam/build/template.yaml \
  --s3-bucket essentials-awss3lambdaartifactsbucket-x29ftznj6pqw \
  --output-template-file .aws-sam/build/cfn-cr-oidc-idp.yaml

aws s3 cp .aws-sam/build/cfn-cr-oidc-idp.yaml s3://bootstrap-awss3cloudformationbucket-19qromfd235z9/cfn-cr-oidc-idp/master/
```

## Install Lambda into AWS
Create the following [sceptre](https://github.com/Sceptre/sceptre) file

config/prod/cfn-cr-oidc-idp.yaml
```yaml
template_path: "remote/cfn-cr-oidc-idp.yaml"
stack_name: "cfn-cr-oidc-idp"
stack_tags:
  Department: "Platform"
  Project: "Infrastructure"
  OwnerEmail: "it@sagebase.org"
hooks:
  before_launch:
    - !cmd "curl https://s3.amazonaws.com/bootstrap-awss3cloudformationbucket-19qromfd235z9/cfn-cr-oidc-idp/master/cfn-cr-oidc-idp.yaml --create-dirs -o templates/remote/cfn-cr-oidc-idp.yaml"
```

Install the lambda using sceptre:
```shell script
sceptre --var "profile=my-profile" --var "region=us-east-1" launch prod/cfn-cr-oidc-idp.yaml
```

## Inspiration

This project was inspired by the [cfn-identity-provider][3] by [Colin Panisset][4]
of [Cevo][5] which provides a similar function but for the SAML identity provider

[1]: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html
[2]: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-lambda-function-code.html#cfn-lambda-function-code-zipfile
[3]: https://github.com/cevoaustralia/cfn-identity-provider
[4]: https://github.com/nonspecialist
[5]: https://cevo.com.au/
