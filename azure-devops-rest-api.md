# Azure DevOps Services REST API Reference

Welcome to the Azure DevOps Services/Azure DevOps Server REST API Reference.

Representational State Transfer (REST) APIs are service endpoints that support sets of HTTP operations (methods), which provide create, retrieve, update, or delete access to the service's resources. This guide covers:

* The basic components of a REST API request/response pair
* Creating and sending a REST request, and handling the response

> Most REST APIs are accessible through client libraries, which can greatly simplify your client code.

## Components of a REST API Request/Response Pair

A REST API request/response pair consists of five components:

1. **The request URI**, in the following form: 
   ```
   VERB https://{instance}[/{team-project}]/_apis[/{area}]/{resource}?api-version={version}
   ```
   
   * **instance**: The Azure DevOps Services organization or server:
     * Azure DevOps Services: `dev.azure.com/{organization}`
     * TFS: `{server:port}/tfs/{collection}` (default port is 8080)
   * **resource path**: Takes the form `_apis/{area}/{resource}`, e.g., `_apis/wit/workitems`
   * **api-version**: Required for all requests, format: `{major}.{minor}[-{stage}[.{resource-version}]]`
     * Examples: `api-version=1.0`, `api-version=1.2-preview`, `api-version=2.0-preview.1`

2. **HTTP request message header** fields:
   * Required HTTP method (GET, HEAD, PUT, POST, PATCH)
   * Optional additional header fields (e.g., Authorization)

3. **Optional HTTP request message body** fields:
   * Used for POST or PUT operations
   * Should specify Content-type in the request header

4. **HTTP response message header** fields:
   * HTTP status code (2xx success, 4xx/5xx error)
   * Optional additional header fields (e.g., Content-type)

5. **Optional HTTP response message body** fields:
   * Typically returned in structured format like JSON or XML

## Authentication

There are several ways to authenticate with Azure DevOps Services/TFS:

| Type of application | Description | Example | Authentication mechanism | Code samples |
|---------------------|-------------|---------|--------------------------|--------------|
| Interactive client-side | Client app with user interaction | Console app enumerating projects | [Microsoft Authentication Library (MSAL)](https://learn.microsoft.com/en-us/azure/active-directory/develop/msal-overview) | [sample](https://github.com/microsoft/azure-devops-auth-samples/tree/master/ManagedClientConsoleAppSample) |
| Interactive JavaScript | GUI-based JavaScript app | AngularJS single page app | [MSAL](https://learn.microsoft.com/en-us/azure/active-directory/develop/reference-v2-libraries) | [sample](https://github.com/microsoft/azure-devops-auth-samples/tree/master/JavascriptWebAppSample) |
| Non-interactive client-side | Headless text-only client app | Console app displaying bugs | Device Profile | [sample](https://github.com/Microsoft/vsts-auth-samples/tree/master/DeviceProfileSample) |
| Interactive web | GUI-based web application | Custom web dashboard | [OAuth](https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth) | [sample](https://github.com/Microsoft/vsts-auth-samples/tree/master/OAuthWebSample) |
| TFS application | App using Client OM library | TFS extension for bug dashboards | [Client Libraries](https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/dotnet-client-libraries) | [sample](https://github.com/Microsoft/vsts-auth-samples/tree/master/ClientLibraryConsoleAppSample) |

### Using Personal Access Tokens

Basic authentication with personal access tokens (PATs) is a simple approach:

```
curl -u {username}:{personalaccesstoken} https://dev.azure.com/{organization}/_apis/projects?api-version=2.0
```

Or without including the token in the command:

```
curl -u {username} https://dev.azure.com/{organization}/_apis/projects?api-version=2.0
```

You can also provide the PAT through an HTTP header by first converting it to a Base64 string:

```
Authorization: Basic BASE64PATSTRING
```

Here's a C# example using HttpClient:

```csharp
public static async void GetProjects()
{
    try
    {
        var personalaccesstoken = "PAT_FROM_WEBSITE";

        using (HttpClient client = new HttpClient())
        {
            client.DefaultRequestHeaders.Accept.Add(
                new System.Net.Http.Headers.MediaTypeWithQualityHeaderValue("application/json"));

            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic",
                Convert.ToBase64String(
                    System.Text.ASCIIEncoding.ASCII.GetBytes(
                        string.Format("{0}:{1}", "", personalaccesstoken))));

            using (HttpResponseMessage response = await client.GetAsync(
                        "https://dev.azure.com/{organization}/_apis/projects"))
            {
                response.EnsureSuccessStatusCode();
                string responseBody = await response.Content.ReadAsStringAsync();
                Console.WriteLine(responseBody);
            }
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }
}
```

For TFS, use: `{server:port}/tfs/{collection}` (default port 8080, default collection is `DefaultCollection`):

```
curl -u {username}:{personalaccesstoken} https://{server}:8080/tfs/DefaultCollection/_apis/projects?api-version=2.0
```

## Processing the Response

A successful response will be returned in JSON format (with a few exceptions like Git blobs).

Example response:

```json
{
    "value": [
        {
            "id": "eb6e4656-77fc-42a1-9181-4c6d8e9da5d1",
            "name": "Fabrikam-Fiber-TFVC",
            "url": "https://dev.azure.com/fabrikam-fiber-inc/_apis/projects/eb6e4656-77fc-42a1-9181-4c6d8e9da5d1",
            "description": "TeamFoundationVersionControlprojects",
            "collection": {
                "id": "d81542e4-cdfa-4333-b082-1ae2d6c3ad16",
                "name": "DefaultCollection",
                "url": "https: //dev.azure.com/fabrikam-fiber-inc/_apis/projectCollections/d81542e4-cdfa-4333-b082-1ae2d6c3ad16",
                "collectionUrl": "https: //dev.azure.com/fabrikam-fiber-inc/DefaultCollection"
            },
            "defaultTeam": {
                "id": "66df9be7-3586-467b-9c5f-425b29afedfd",
                "name": "Fabrikam-Fiber-TFVCTeam",
                "url": "https://dev.azure.com/fabrikam-fiber-inc/_apis/projects/eb6e4656-77fc-42a1-9181-4c6d8e9da5d1/teams/66df9be7-3586-467b-9c5f-425b29afedfd"
            }
        },
        {
            "id": "6ce954b1-ce1f-45d1-b94d-e6bf2464ba2c",
            "name": "Fabrikam-Fiber-Git",
            "url": "https://dev.azure.com/fabrikam-fiber-inc/_apis/projects/6ce954b1-ce1f-45d1-b94d-e6bf2464ba2c",
            "description": "Gitprojects",
            "collection": {
                "id": "d81542e4-cdfa-4333-b082-1ae2d6c3ad16",
                "name": "DefaultCollection",
                "url": "https://dev.azure.com/fabrikam-fiber-inc/_apis/projectCollections/d81542e4-cdfa-4333-b082-1ae2d6c3ad16",
                "collectionUrl": "https://dev.azure.com/fabrikam-fiber-inc/DefaultCollection"
            },
            "defaultTeam": {
                "id": "8bd35c5e-30bb-4834-a0c4-d576ce1b8df7",
                "name": "Fabrikam-Fiber-GitTeam",
                "url": "https://dev.azure.com/fabrikam-fiber-inc/_apis/projects/6ce954b1-ce1f-45d1-b94d-e6bf2464ba2c/teams/8bd35c5e-30bb-4834-a0c4-d576ce1b8df7"
            }
        }
    ],
    "count": 2
}
```

## API and Server Version Mapping

REST API versions are compatible with the Server version listed and newer versions. Always use the latest release version with Azure DevOps Service.

| Server Version             | REST API Version | Build Version              |
|----------------------------|------------------|----------------------------|
| Azure DevOps Server vNext  | 7.2              |                            |
| Azure DevOps Server 2022.1 | 7.1              | versions >= 19.225.34309.2 |
| Azure DevOps Server 2022   | 7.0              | versions >= 19.205.33122.1 |
| Azure DevOps Server 2020   | 6.0              | versions >= 18.170.30525.1 |
| Azure DevOps Server 2019   | 5.0              | versions >= 17.143.28621.4 |
| TFS 2018 Update 3          | 4.1              | versions >= 16.131.28106.2 |
| TFS 2018 Update 2          | 4.1              | versions >= 16.131.27701.1 |
| TFS 2018 Update 1          | 4.0              | versions >= 16.122.27409.2 |
| TFS 2018 RTW               | 4.0              | versions >= 16.122.27102.1 |
| TFS 2017 Update 2          | 3.2              | versions >= 15.117.26714.0 |
| TFS 2017 Update 1          | 3.1              | versions >= 15.112.26301.0 |
| TFS 2017 RTW               | 3.0              | versions >= 15.105.25910.0 |
| TFS 2015 Update 4          | 2.3              | versions >= 14.114.26403.0 |
| TFS 2015 Update 3          | 2.3              | versions >= 14.102.25423.0 |
| TFS 2015 Update 2          | 2.2              | versions >= 14.95.25122.0  |
| TFS 2015 Update 1          | 2.1              | versions >= 14.0.24712.0   |
| TFS 2015 RTW               | 2.0              | versions >= 14.0.23128.0   |

## Client Libraries

Client libraries are available for these REST APIs:

* [.NET](https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/dotnet-client-libraries)
* [Go](https://github.com/microsoft/azure-devops-go-api)
* [Node.js](https://github.com/Microsoft/azure-devops-node-api)
* [Python](https://github.com/Microsoft/azure-devops-python-api)
* [Swagger 2.0](https://github.com/microsoft/azure-devops-swagger-tools)
* [Web Extensions SDK](https://github.com/Microsoft/azure-devops-extension-api)

## Related Resources

* [Authentication guidance](https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/authentication-guidance)
* [Samples](https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/rest/samples)
* [REST API for older versions (Before 4.1)](https://learn.microsoft.com/en-us/previous-versions/azure/devops/integrate/previous-apis/overview)

Source: [Azure DevOps Services REST API Reference](https://learn.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-7.2) 