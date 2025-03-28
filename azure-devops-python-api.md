# Azure DevOps Python API

This repository contains Python APIs for interacting with and managing Azure DevOps. These APIs power the Azure DevOps Extension for Azure CLI.

## Installation

```bash
pip install azure-devops
```

## Getting Started

To use the API, establish a connection using a personal access token and the URL to your Azure DevOps organization. Then get a client from the connection and make API calls.

```python
from azure.devops.connection import Connection
from msrest.authentication import BasicAuthentication
import pprint

# Fill in with your personal access token and org URL
personal_access_token = 'YOURPAT'
organization_url = 'https://dev.azure.com/YOURORG'

# Create a connection to the org
credentials = BasicAuthentication('', personal_access_token)
connection = Connection(base_url=organization_url, creds=credentials)

# Get a client (the "core" client provides access to projects, teams, etc)
core_client = connection.clients.get_core_client()

# Get the first page of projects
get_projects_response = core_client.get_projects()
index = 0
while get_projects_response is not None:
    for project in get_projects_response.value:
        pprint.pprint("[" + str(index) + "] " + project.name)
        index += 1
    if get_projects_response.continuation_token is not None and get_projects_response.continuation_token != "":
        # Get the next page of projects
        get_projects_response = core_client.get_projects(continuation_token=get_projects_response.continuation_token)
    else:
        # All projects have been retrieved
        get_projects_response = None
```

## API Documentation

This Python library provides a thin wrapper around the Azure DevOps REST APIs. See the [Azure DevOps REST API reference](https://learn.microsoft.com/en-us/rest/api/azure/devops/) for details on calling different APIs.

## Samples

Learn how to call different APIs by viewing the samples in the [Microsoft/azure-devops-python-samples](https://github.com/Microsoft/azure-devops-python-samples) repository.

## Available Clients

The Python SDK provides multiple clients for interacting with specific Azure DevOps services:

- Core Client (`get_core_client()`): For working with projects, teams, etc.
- Git Client (`get_git_client()`): For working with repositories, pull requests, etc.
- Work Item Tracking Client (`get_work_item_tracking_client()`): For working with work items
- Build Client (`get_build_client()`): For working with build definitions and builds
- Release Client (`get_release_client()`): For working with release definitions
- Task Agent Client (`get_task_agent_client()`): For working with agent pools and agents
- Wiki Client (`get_wiki_client()`): For working with wikis

## Contributing

This project welcomes contributions and suggestions. Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit [https://cla.microsoft.com](https://cla.microsoft.com).

## License

This project is licensed under the MIT License.

Source: [Microsoft/azure-devops-python-api](https://github.com/Microsoft/azure-devops-python-api) 