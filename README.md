# Bitbucket Pipelines Pipe: AWS EKS Helm Deploy

Deploy Helm charts to AWS EKS

## YAML Definition

Add the following snippet to the script section of your `bitbucket-pipelines.yml` file:

```yaml
- pipe: docker://richarddsmith/aws-eks-helm-deploy:1.2.0
  variables:
    AWS_ACCESS_KEY_ID: "<string>"
    AWS_SECRET_ACCESS_KEY: "<string>"
    CLUSTER_NAME: "<string>"
    CHART: "<string>"
```

## Variables

| Variable                  | Usage                                                                  |
| ------------------------- | ---------------------------------------------------------------------- |
| AWS_REGION                | AWS Region. Default: `eu-central-1`. |
| AWS_ACCESS_KEY_ID (*)     | AWS Access Key ID |
| AWS_SECRET_ACCESS_KEY (*) | AWS Secret Access Key |
| ROLE_ARN                  | AWS IAM Role to assume when access EKS |
| SESSION_NAME              | AWS STS Session name |
| CLUSTER_NAME (*)          | Name of the AWS EKS cluster |
| CHART (*)                 | Path or name of the chart which should be deployed |
| RELEASE_NAME              | Name of the helm release |
| NAMESPACE                 | Target Kubernetes namespace of the deployment. Default: `kube-public`. |
| CREATE_NAMESPACE          | Boolean. Will instruct helm to create a namespace if missing. Default: `false`. |
| SET                       | List of values which should be used as --set argument for Helm |
| VALUES                    | Local values YAML files which should be passed to Helm (--values) |
| DEBUG                     | Debug. Default: `false`. |
| WAIT                      | Wait until application is ready. Default: `false`. |
| DEBUG                     | Debug. Default: `false`. |
| UNINSTALL                 | Uninstall the release from the given namespace. Default: `false` |
| INSTALL_SUBCHARTS         | Run `helm dependency update` to pull in the latest subcharts based on `Charts.yaml`. Default: `false` |

_(*) = required variable._

## Prerequisites

## Examples

Basic example:

```yaml
script:
  - pipe: docker://richarddsmith/aws-eks-helm-deploy:1.2.0
    variables:
      NAME: "foobar"
```

Advanced example which uses AWS SecretsManager and different AWS IAM Roles

```yaml

script:
  - step:
      name: Deploy
      image: amazon/aws-cli
      deployment: Development
      caches:
        - docker
      script:
        - yum install -y -q jq
        - aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID --profile default
        - aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY --profile default
        - aws configure set region eu-central-1 --profile default
        - aws configure set role_arn $VAULT_ROLE_ARN --profile vault
        - aws configure set source_profile default --profile vault
        - aws configure set region eu-central-1 --profile vault
        - aws secretsmanager get-secret-value --secret-id application/secret --profile vault | jq -r ".SecretString" > secrets.yaml
  - pipe: docker://richarddsmith/aws-eks-helm-deploy:1.1.3
    variables:
      AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
      AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
      ROLE_ARN: $KUBERNETES_USER_ROLE_ARN
      CLUSTER_NAME: a-cluster-name
      CHART: path-to-helm-chart
      RELEASE_NAME: my-example-release
      NAMESPACE: default
      SET: [
        'replicaCount=3',
        'image.version=1.0.2-${BITBUCKET_BUILD_NUMBER}',
        'env.foo_from_repository_or_deployment_variable=${BAR}',
      ]
      VALUES: [
        secrets.yaml
      ]

```

## Support
If you’d like help with this pipe, or you have an issue or feature request, let me know.
The pipe is maintained by yves.vogl@adesso.de

If you’re reporting an issue, please include:

- the version of the pipe
- relevant logs and error messages
- steps to reproduce
