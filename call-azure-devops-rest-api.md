# Calling the Azure DevOps REST API

The Azure DevOps REST APIs follow a common pattern:

```
VERB https://{instance}/{collection}/{team-project}/_apis/{area}/{resource}?api-version={version}
```

> **Tip**: Always include an API version in every request to avoid unexpected changes in the API that could break your integration.

## Azure DevOps Services

For Azure DevOps Services, `instance` is `dev.azure.com/{organization}` and `collection` is `DefaultCollection`:

```
VERB https://dev.azure.com/{organization}/_apis/{area}/{resource}?api-version={version}
```

Example of getting a list of projects:

```bash
curl -u {username}:{personalaccesstoken} https://dev.azure.com/{organization}/_apis/projects?api-version=2.0
```

## Azure DevOps Server

For Azure DevOps Server, `instance` is `{server:port}` (default port for non-SSL is 8080):

```
curl -u {username}:{personalaccesstoken} https://{server}/DefaultCollection/_apis/projects?api-version=2.0
```

## Authentication

### Personal Access Tokens (PATs)

You can provide the PAT through an HTTP header by converting it to a Base64 string:

```
Authorization: Basic BASE64PATSTRING
```

Example using C# and HttpClient:

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

            using (HttpResponseMessage response = client.GetAsync(
                        "https://dev.azure.com/{organization}/_apis/projects").Result)
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

> **Important**: While personal access tokens (PATs) are used in many examples for simplicity, they are not recommended for production applications. Consider using more secure authentication mechanisms.

## Common Response Codes

| Code | Description |
|------|-------------|
| 200 | Success, and there's a response body. |
| 201 | Success, and the resource was created. You'll get a response body. Most `POST` methods return 201. |
| 204 | Success, and there's no response body. For example, you get this response when you delete a resource. |
| 400 | The parameters in the URL or in the request body aren't valid. |
| 401 | Authentication failed. Often because of a missing or malformed Authorization header. |
| 403 | The authenticated user doesn't have permission to do the operation. |
| 404 | The resource doesn't exist, or the authenticated user doesn't have permission to see that it exists. |
| 409 | Conflict between the request and the state of the data on the server. |

## Cross-Origin Resource Sharing (CORS)

Azure DevOps Services supports CORS, enabling JavaScript code from other domains to make Ajax requests to Azure DevOps Services REST APIs:

```javascript
$(document).ready(function() {
    $.ajax({
        url: 'https://dev.azure.com/fabrikam/_apis/projects?api-version=1.0',
        dataType: 'json',
        headers: {
            'Authorization': 'Basic ' + btoa("" + ":" + myPatToken)
        }
    }).done(function(results) {
        console.log(results.value[0].id + " " + results.value[0].name);
    });
});
```

## API Versioning

Azure DevOps REST APIs use versioning to ensure applications continue to work as APIs evolve.

### Guidelines

* Specify the API version with every request (required)
* Format: `{major}.{minor}[-{stage}[.{resource-version}]]` (e.g., `1.0`, `1.2-preview`, `2.0-preview.1`)
* Preview APIs are deprecated once the released version is available

### Specifying the Version

You can specify the API version in the header or as a query parameter:

HTTP request header:
```
Accept: application/json;api-version=1.0
```

Query parameter:
```
GET https://dev.azure.com/{organization}/_apis/{area}/{resource}?api-version=1.0
```

Source: [Get started with the REST APIs](https://learn.microsoft.com/en-us/azure/devops/integrate/how-to/call-rest-api?view=azure-devops) 