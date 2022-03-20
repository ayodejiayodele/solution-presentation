# API Documentation

Please keep your source files here, and remember to add some good documentation 😉

[![Board Status](https://dev.azure.com/ayotoday/90fb8fe1-1601-4c43-ac1b-11de1158286a/d2da950a-6a9d-41f4-aecc-3647caa4b755/_apis/work/boardbadge/12665d10-715f-482a-b8d8-d98dd6214fdd?columnOptions=1)](https://dev.azure.com/ayotoday/90fb8fe1-1601-4c43-ac1b-11de1158286a/_boards/board/t/d2da950a-6a9d-41f4-aecc-3647caa4b755/Microsoft.RequirementCategory/)

[![Deploy To Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fintegrityplus%2Fsolution-presentation%2Fuser%2Fayodejiayodele%2Fapi-logic%2Fsrc%2Ftemplate.json)

# Architecture
## General overview of solution architecture
Two Azure service components were used in setting up the API
| Component       | Description |
| --------------- | ----------- |
| Azure Logic App | Primary API PaaS using containerised runtime |
| Azure Key Vault | Vault for storing secrets and personal access token to connect to the GitHub organization |

<img alt="High level overview of Web API" src="images/high-level-architecture.png" width="600"></img>

## API Logic Flow
The web API depends on the GitHub [REST API](https://docs.github.com/en/rest) (branches and issues) for retrieving and sending data to/from the GitHub organization. This makes use of [Basic Authentication](https://docs.github.com/en/rest/overview/other-authentication-methods#via-oauth-and-personal-access-tokens) using OAuth tokens. The authentication token is stored as a secret in the key vault during [deployment](#how-to-deploy).


> To successfully make REST API calls to GitHub, you need to [generate a Personal Access Token (PAT)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token#creating-a-token) for the user who is an owner of the organization.
> GitHub has discontinued password authentication to the API starting on November 13, 2020 for all GitHub.com accounts, including those on a GitHub Free, GitHub Pro, GitHub Team, or GitHub Enterprise Cloud plan. You must now authenticate to the GitHub API with an API token, such as an OAuth access token, GitHub App installation access token, or personal access token.

The diagram below shows the logic flow of the web API.

<img alt="Process flow of API" src="images/flow-diagram.png" width="600"></img>

### Trigger event
The [webhook](/README.md#Implement-default-branch-protection), as mentioned in the solution approach, generates a payload whenever a branch or tag is created on any repository in the organization. The API App is configured with a manual trigger, listening to a POST call, and the JSON schema is matched with what is expected to be received.

The API then stores three important properties from the request body in variables.
| Property                     | Description |
| ---------------------------- |
| body.repository.branches_url | The API URL to manage branches of the repo. Removed the trailing `{/branch}` |
| body.master_branch           | The default branch name of the repo. Appended this to the branches URL later in the code |
| body.repository.issues_url   | The API URL to manage issues of the repo. Removed the trailing `{/number}` |

<img alt="API manual trigger" src="images/manual-trigger.png" width="600"></img>


### Validation
The request header comes a header property `X-GitHub-Event`, which indicates what organization event fired the webhook. In the case of branch creation, the value of this header is expected to be `create`. In addition to validating the event, the API also verifies if this is the branch created matches the default branch name for the repo i.e. `body.ref` should be the same as `body.master_branch`.

### Processing
Once validated to be true for both conditions, the API retrieves the personal access token from the key vault and passes this along with the username as Basic Authentication in subsequent API calls.


## How to Deploy
This makes use of Azure ARM template deployment, using a [template file](template.json). For other deployment methods please visit [deployment operations on Azure](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deploy-portal).

[![Deploy To Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fintegrityplus%2Fsolution-presentation%2Fuser%2Fayodejiayodele%2Fapi-logic%2Fsrc%2Ftemplate.json)

### Pre-requiresites for deployment
 - [x] An Azure subscription with at least `Contributor` role on the existing resource group or permissions to create a new one.
 - [x] A personal access token, copy temporarily to a text file.
 
 ### Parameters required
 | Parameter Name        | Type         | Description |
 | --------------------- | ------------ | --- |
 | Key Vault Name        | string       | Name of the key vault resource. |
 | Logic App Name        | string       | Name of the logic app resource. |
 | Connection Name       | string       | Name of the connection resource connecting the logic app to the key vault. |
 | Personal Access Token | secureString | Personal access token copied earlier. |

 ### Post-deployment steps
 Once the logic app is ready, open the logic app resource and click on Logic App designer. Then, click on the _When HTTP request is received_ trigger to expand it. Then copy the `HTTP POST URL` text into a temporary text file. Then create a webhook with the properties specified in the [solution approach](/README.md/#Implement-default-branch-protection).

<img alt="Expanded trigger" src="images/trigger-expanded.png" width="600"></img>
