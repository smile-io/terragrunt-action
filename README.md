# terragrunt-action

A GitHub Action for installing and running Terragrunt


## Requirements
- jq
- tfenv
- tgswitch
## Inputs

Supported GitHub action inputs:

| Input Name | Description                                       | Required | Example values |
|:-----------|:--------------------------------------------------|:--------:|:--------------:|
| tf_version | Terraform version to be used in Action execution  |  `true`  |     1.4.6      | 
| tg_version | Terragrunt version to be user in Action execution |  `true`  |     0.50.8     |
| tg_dir     | Directory in which Terragrunt will be invoked     |  `true`  |      work      |
| tg_command | Terragrunt command to execute                     |  `true`  |   plan/apply   |
| tg_comment | Add comment to Pull request with execution output | `false`  |      0/1       |

## Environment Variables

Supported environment variables:

| Input Name            | Description                                                                                                 | 
|:----------------------|:------------------------------------------------------------------------------------------------------------|
| GITHUB_TOKEN          | GitHub token used to add comment to Pull request                                                            |
| TF_LOG                | Log level for Terraform                                                                                     |
| TF_VAR_name           | Define custom variable name as inputs                                                                       |

## Outputs

Outputs of GitHub action:

| Input Name          | Description                     |
|:--------------------|:--------------------------------|
| tg_action_exit_code | Terragrunt exit code            |
| tg_action_output    | Terragrunt output as plain text |

## Usage

Example definition of Github Action workflow:

```yaml
name: 'Terragrunt GitHub Actions'
on:
  - pull_request

env:
  tf_version: '1.4.6'
  tg_version: '0.46.3'
  working_dir: 'project'

jobs:
  checks:
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout'
        uses: actions/checkout@main

      - name: Check terragrunt HCL
        uses: smile-io/terragrunt-action@v1
        with:
          tf_version: ${{ env.tf_version }}
          tg_version: ${{ env.tg_version }}
          tg_dir: ${{ env.working_dir }}
          tg_command: 'hclfmt --terragrunt-check --terragrunt-diff'

  plan:
    runs-on: ubuntu-latest
    needs: [ checks ]
    steps:
      - name: 'Checkout'
        uses: actions/checkout@main

      - name: Plan
        uses: smile-io/terragrunt-action@v1
        with:
          tf_version: ${{ env.tf_version }}
          tg_version: ${{ env.tg_version }}
          tg_dir: ${{ env.working_dir }}
          tg_command: 'plan'

  deploy:
    runs-on: ubuntu-latest
    needs: [ plan ]
    environment: 'prod'
    if: github.ref == 'refs/heads/main'
    steps:
      - name: 'Checkout'
        uses: actions/checkout@main

      - name: Deploy
        uses: smile-io/terragrunt-action@v1
        with:
          tf_version: ${{ env.tf_version }}
          tg_version: ${{ env.tg_version }}
          tg_dir: ${{ env.working_dir }}
          tg_command: 'apply'
```
