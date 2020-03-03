# jira-github-action-integration-demo 

This demo has been made to showcase how to integrate CI/CD dev metrics(build/deploy) between a Github Workflow and Jira Cloud with three Github Actions:

- [Jira Extract Issue Keys](https://github.com/h3demo/jira-github-action-integration-demo#jira-extract-issue-keys)
- [Jira Upload Build Info](https://github.com/h3demo/jira-github-action-integration-demo#jira-upload-build-info)
- [Jira Upload Deployment Info](https://github.com/h3demo/jira-github-action-integration-demo#jira-upload-deployment-info)

To summarize the actions taken in this demo pipeline, Jira Extract Issue Keys extracts the values in GitHub commits that correspond to a project issue key, example: A-1, PROJECT-2, PROJECT-3. If the project is building and has a key in the commit message, then its build informaiton is sent to any matching project issue keys that exist in a linked Jira Cloud via Jira Upload Build Info. If the project is deploying and has a key in the commit message, then its deployment information is sent to any matching project issue keys that exist in a connected Jira Cloud via Jira Upload Deployment Info. 

## Requirements

For Jira Extract Issue Keys, you must be able to use push and/or pull requests in GitHub.

For Jira Upload Build Info and Jira Upload Deployment Info, you must have the `GitHub for Jira` app installed on the Jira Cloud instance you plan to use, the `Jira Software + GitHub` installed on your GitHub account, and have your GitHub account connected to the Jira Cloud instance.

Jira Upload Build Info and Jira Upload Deployment Info also require OAuth credentails from Jira Cloud for the pipeline in order to send trusted information to and from GitHub and Jira Cloud. For more informaiton on how to set up OAuth credentials, please see Action Secrets in the [Jira Upload Build Info](https://github.com/HighwayThree/jira-upload-build-info#action-secrets) or [Jira Upload Deployment Info](https://github.com/HighwayThree/jira-upload-deployment-info#action-secrets)'s README files.

There should be at least two secrets in your GitHub account, one for the client ID and one for the client secret. For more details, please read the [Jira Upload Build Info](https://github.com/h3demo/jira-github-action-integration-demo#job-specific-variables-1) and [Jira Upload Deployment Info](https://github.com/h3demo/jira-github-action-integration-demo#job-specific-variables-2) job specific variables for examples of the required inputs.

## GitHub Actions

### [Jira Extract Issue Keys:](https://github.com/HighwayThree/jira-extract-issue-keys)

**This Github Action parses all the possible issue keys from the commit message.**

###### Environment variables

```
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

###### Optional variables

```
uses: HighwayThree/jira-extract-issue-keys@master
with:
  is-pull-request: ${{ github.event_name == 'pull_request' }}
  parse-all-commits: ${{ github.event_name == 'push' }}
  commit-message: 'EXAMPLE-1 message'
```
- `is-pull-request` - is true if the GitHub event is a pull request. Default is false.
- `parse-all-commits` - is true if the GitHub event is a push. Default is false.
- `commit-message` - commit message to be parsed for jira keys. Default is the user's commit message.

### [Jira Upload Build Info:](https://github.com/HighwayThree/jira-upload-build-info)

**This Github Action uploads build information to specified Jira issues belonging to a connected Jira Cloud instance via the Jira Software REST API.**

###### Environment variables
```
env:
  GITHUB_RUN_ID: ${{secrets.GITHUB_RUN_ID}}
  GITHUB_RUN_NUMBER: ${{secrets.GITHUB_RUN_NUMBER}}
```

###### Job specific variables

```
uses: HighwayThree/jira-upload-build-info@master
with:
  client-id: '${{ secrets.CLIENT_ID }}'
  client-secret: '${{ secrets.CLIENT_SECRET }}'
  cloud-instance-base-url: '${{ secrets.CLOUD_INSTANCE_BASE_URL }}'
  issue-keys: "${{ steps.jira_keys.outputs.jira-keys }}"
```

- `client-id` - Access token found in OAth credentials of your Jira Cloud website. `'${{ secrets.CLIENT_ID }}'` can be changed to match a secret's name in your GitHub's secrets. It is not recommended to hard-code your Client ID in the pipeline.
- `client-secret` - Access token found in OAth credentials of your Jira Cloud website. `'${{ secrets.CLIENT_SECRET }}'` can be changed to match a secret's name in your GitHub's secrets. It is not recommended to hard-code your Client Secret in the pipeline.
- `cloud-instance-base-url` - The base URL of your connected Jira Cloud. In this example it is stored as a GitHub secret, but another example could be `cloud-instance-base-url: 'https://example.atlassian.net'`
- `issue-keys` - Key values that correspond with Jira issues of the connected Jira Cloud. These are gathered in the Jira Extract Issue Keys GitHub Action.

###### Optional variables

This pipeline demo does not contain any optional variables. Not manually setting optional values sets them to the default values seen in the code below. For more information about the optional variabes, please see [Jira Upload Build Info's Optional Values](https://github.com/HighwayThree/jira-upload-build-info#optional).

```
with:
  pipeline-id: '${{ github.repository }} ${{ github.workflow }}'
  build-number: ${{ github.run_number }}
  build-display-name: 'Workflow: ${{ github.workflow }} (#${{ github.run_number }})'
  build-state: "${{ env.BUILD_STATE }}"
  build-url: '${{github.event.repository.url}}/actions/runs/${{github.run_id}}'
  update-sequence-number: '${{ github.run_id }}'
  last-updated: '${{github.event.head_commit.timestamp}}'
  commit-id: '${{ github.sha }}'
  repo-url: '${{ github.event.repository.url }}'
  build-ref-url: '${{ github.event.repository.url }}/actions/runs/${{ github.run_id }}'
```

### [Jira Upload Deployment Info:](https://github.com/HighwayThree/jira-upload-deployment-info)

**This Github Action uploads deployment information to specified Jira issues belonging to a connected Jira Cloud instance via the Jira Software REST API.**

###### Environment variables
```
env:
  GITHUB_RUN_ID: ${{secrets.GITHUB_RUN_ID}}
  GITHUB_RUN_NUMBER: ${{secrets.GITHUB_RUN_NUMBER}}
```

###### Job specific variables
```
uses: HighwayThree/jira-upload-deployment-info@master
with:
  client-id: '${{ secrets.CLIENT_ID }}'
  client-secret: '${{ secrets.CLIENT_SECRET }}'
  cloud-instance-base-url: '${{ secrets.CLOUD_INSTANCE_BASE_URL }}'
  issue-keys: "${{ steps.jira_keys.outputs.jira-keys }}"
  display-name: "Deployment Number 1"
  description: "Test Deployment"
  last-updated: '${{github.event.head_commit.timestamp}}'
  label: 'Test Deployment Label'
  state: '${{env.DEPLOY_STATE}}'
  environment-id: 'Test'
  environment-display-name: 'Test'
  environment-type: 'testing'
```

- `client-id` - Access token found in OAth credentials of your Jira Cloud website. `'${{ secrets.CLIENT_ID }}'` can be changed to match a secret in your GitHub's secrets. It is not recommended to hard-code your Client ID in the pipeline.
- `client-secret` - Access token found in OAth credentials of your Jira Cloud website. `'${{ secrets.CLIENT_SECRET }}'` can be changed to match a secret in your GitHub's secrets. It is not recommended to hard-code your Client Secret in the pipeline.
- `cloud-instance-base-url` - The base URL of your connected Jira Cloud. In this example it is stored in a GitHub secret, but another example could be 'https://<span></span>example.atlassian.net'
- `issue-keys` - Key values that correspond with Jira issues of the connected Jira Cloud. These are gathered in the Jira Extract Issue Keys GitHub Action.
- `display-name` - The title for the deployment. It can be any string.
- `description` - Provides a description of the deployment. It can be any string.
- `last-updated` - A timestamp is created with the format yyyy-mm-dd'T'HH:MM:ss'Z'. Input should look something like 2020-01-01T00:01:00-08:00
- `label` - Provides a label for the deployment. It can be any string.
- `state` - Gives a state for the deployment. For example: CI
- `environment-id` - Provides an ID for the environment. It can be any string.
- `environment-display-name` - The title for the environment. It can be any string.
- `environment-type` - The type of environment. It can be any string.

###### Optional variables

This pipeline demo does not contain any optional variables. Not manually setting optional values sets them to the default values seen in the code below. For more information about the optional variabes, please see [Jira Upload Deployment Info's Optional Values](https://github.com/HighwayThree/jira-upload-deployment-info#optional).

```
with:
  deployment-sequence-number: '${{ github.run_id }}'
  update-sequence-number: '${{ github.run_id }}'
  url: "${{github.event.repository.url}}/actions/runs/${{github.run_id}}"
  description: "Test Deployment"
  pipeline-id: '${{ github.repository }} ${{ github.workflow }}'
  pipeline-display-name: 'Workflow: ${{ github.workflow }} (#${{ github.run_number }})'
  pipeline-url: '${{github.event.repository.url}}/actions/runs/${{github.run_id}}'
```

## Jobs in Demo Pipeline

The pipeline activates when there is a push on branches:
- master
- release/1.0.0
- feature/*

The pipeline also activates when there is a pull request on branches:
- bugfix/*
- feature/*
- hotfix/*
- release/*

#### build_and_test

This job runs during push and pull requests of the branches given above. If the output of Jira Extract Issue Keys is not empty, then the build information is pushed to the Jira Cloud issue(s). 

#### deploy_to_test

This job only runs during push requests to the release/1.0.0 branch. If the output of Jira Extract Issue Keys is not empty, then the deployment information is pushed to the Jira Cloud issue(s). 

#### deploy_to_prod

This job only runs during push requests to the master branch. Like with deploy_to_test, this job sends deployment information to Jira Cloud issue(s) specified in the commit message if the Jira Extract Issue Keys output is not empty.

