# Inspector Gadget

Inspector Gadget is a simple web app that you can use to get a lot of info on the web server where it's running (the *inspector*) and perform additional tasks (the *gadgets*) from there.

There are two versions that you can choose from:

- The full-featured .NET Core based [web app](WebApp) that can run on Linux or Windows.
  - You can compile the source code and publish the app to your host of choice directly, or build and run it as a container.
  - You can also find the latest published version of the Docker container publicly on **Docker Hub** at **[jelledruyts/inspectorgadget](https://hub.docker.com/r/jelledruyts/inspectorgadget)**.
    - For **Linux**: specify the image name and tag as `jelledruyts/inspectorgadget:latest` or simply `jelledruyts/inspectorgadget`.
    - For **Windows**: specify the image name and tag as `jelledruyts/inspectorgadget:latest-windows`.
- Alternatively, there's also a single [default.aspx](Page/default.aspx) page that you can simply drop in any Windows and ASP.NET Framework based host (like IIS or Azure App Service).
  - You can just copy this file into e.g. the `wwwroot` directory of your site (optionally rename it) and navigate to it directly without any compilation required.
  - This version does not have all the bells and whistles of the full web app (so not all features and configuration settings mentioned below will apply, e.g. it does not have all gadgets and no support for API and Call Chaining), but it can still be a useful drop-in page for basic tasks.

## Gadgets

The following gadgets are available:

| Gadget                     | Purpose                                                                                                                                                                                                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **DNS Lookup**             | Allows you to perform a DNS lookup from the web server.                                                                                                                                                                                    |
| **SQL Connection**         | Allows you to perform a (scalar) query on a SQL Connection from the web server (optionally using an Azure Managed Identity to connect to Azure SQL Database).                                                                              |
| **Azure Managed Identity** | Allows you to request an access token for the [managed identity representing your application](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) (when running on a supported Azure service). |
| **Socket Connection**      | Allows you to perform a raw TCP socket connection from the web server (optionally with a request body and reading back the response).                                                                                                      |
| **Process Run**            | Allows you to run a process on the host and capture the output.                                                                                                                                                                            |
| **Introspector**           | Allows you to perform an inspector request from the web server, returning all information the inspector knows about or only a subset (group) or even single item (key).                                                                    |

## API Access

Each gadget can also be accessed through a REST API:

| Method | Path                              | Parameters                                                      |
| ------ | --------------------------------- | --------------------------------------------------------------- |
| POST   | `/api/dnslookup`                  | `host`                                                          |
| POST   | `/api/httprequest`                | `requestUrl`, `requestHostName`                                 |
| POST   | `/api/sqlconnection`              | `sqlConnectionString`, `sqlQuery`                               |
| POST   | `/api/azuremanagedidentity`       | `resource`                                                      |
| POST   | `/api/socketconnection`           | `requestHostName`, `requestPort`, `requestBody`, `readResponse` |
| POST   | `/api/processrun`                 | `fileName`, `arguments`, `timeoutSeconds`                       |
| GET    | `/api/introspector`               |                                                                 |
| GET    | `/api/introspector/<group>`       |                                                                 |
| GET    | `/api/introspector/<group>/<key>` |                                                                 |
| POST   | `/api/introspector`               | `group`, `key`                                                  |

## Call Chaining

The main purpose for these API's is so that you can perform call chains of requests from one deployed instance of the application to one or more other instances. This makes it easy to set up a multi-tier test environment (e.g. on Kubernetes or other microservice platforms) and perform queries across multiple hops.

> Imagine for example that you deploy a Kubernetes cluster with 4 pods, each of these running the exact same Inspector Gadget container. You can then add an [info message](#configuration) or environment variable like `tier` to each pod indicating which tier they logically represent, e.g. `front-end`, `api-gateway`, `api-1` and `api-2`. With that configuration in place, you can then perform a call chain to hop from the `front-end` to the `api-gateway` and from there to `api-1`.
>
> For example, to validate that the tiers are able to reach each other (e.g. if you've deployed network policies or when you're using a service mesh), you can browse to the Inspector Gadget web page on the `front-end`, and from there perform an **Introspector** request by setting the "Call Chain" to `http://api-gateway http://api-1` (if those would be the URL's of the other pods of course). If you set the requested Introspector "Group" to `Environment` and the "Key" to `tier` (or whatever environment variable you picked), then you can see the simple value returned by each hop to show which tier it represents.
>
> Note that this would work even if only the `front-end` is exposed externally, with the other tiers only accessible from within the cluster, because each call in the chain is done from the previous hop.

## Configuration

The app can be configured with the configuration settings below (using environment variables).

| Setting                                       | Purpose                                                                                                                         |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `BackgroundColor`                             | An HTML/CSS color value to set as the background color for all pages, e.g. to easily distinguish multiple instances of this app |
| `InfoMessage`                                 | An informational message to put at the top of all pages, e.g. to easily distinguish multiple instances of this app              |
| `DefaultCallChainUrls`                        | The default value for the Call Chain URL's (applies to all gadgets)                                                             |
| `DisableAzureManagedIdentity`                 | Allows you to disable the **Azure Managed Identity** gadget                                                                     |
| `DefaultAzureManagedIdentityResource`         | The default value for the Azure Managed Identity gadget resource                                                                |
| `DisableDnsLookup`                            | Allows you to disable the **DNS Lookup** gadget                                                                                 |
| `DefaultDnsLookupHost`                        | The default value for the DNS Lookup gadget host                                                                                |
| `DisableHttpRequest`                          | Allows you to disable the **HTTP Request** gadget                                                                               |
| `DefaultHttpRequestUrl`                       | The default value for the HTTP Request gadget URL                                                                               |
| `DefaultHttpRequestHostName`                  | The default value for the HTTP Request gadget host name                                                                         |
| `DisableIntrospector`                         | Allows you to disable the **Introspector** gadget                                                                               |
| `DefaultIntrospectorGroup`                    | The default value for the Introspector gadget group                                                                             |
| `DefaultIntrospectorKey`                      | The default value for the Introspector gadget key                                                                               |
| `DisableProcessRun`                           | Allows you to disable the **Process Run** gadget                                                                                |
| `DefaultProcessRunFileName`                   | The default value for the Process Run gadget file name                                                                          |
| `DefaultProcessRunArguments`                  | The default value for the Process Run gadget arguments                                                                          |
| `DefaultProcessRunTimeoutSeconds`             | The default value for the Process Run gadget timeout (in seconds)                                                               |
| `DisableSocketConnection`                     | Allows you to disable the **Socket Connection** gadget                                                                          |
| `DefaultSocketConnectionRequestHostName`      | The default value for the Socket Connection gadget request host name                                                            |
| `DefaultSocketConnectionRequestPort`          | The default value for the Socket Connection gadget request port                                                                 |
| `DefaultSocketConnectionRequestBody`          | The default value for the Socket Connection gadget request body                                                                 |
| `DefaultSocketConnectionReadResponse`         | The default value for the Socket Connection gadget setting to read the response                                                 |
| `DisableSqlConnection`                        | Allows you to disable the **SQL Connection** gadget                                                                             |
| `DefaultSqlConnectionSqlConnectionString`     | The default value for the SQL Connection gadget SQL connection string                                                           |
| `DefaultSqlConnectionSqlQuery`                | The default value for the SQL Connection gadget SQL query                                                                       |
| `DefaultSqlConnectionUseAzureManagedIdentity` | The default value for the SQL Connection gadget setting to use the Azure Managed Identity of the app                            |

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
