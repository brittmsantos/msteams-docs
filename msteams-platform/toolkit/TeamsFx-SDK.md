---
title: TeamsFx SDK
author: surbhigupta
description: In this module, learn about TeamsFx SDK, core concepts and code structure, advanced Customization and scenarios
ms.author: v-amprasad
ms.localizationpriority: medium
ms.topic: overview
ms.date: 11/29/2021
---

# TeamsFx SDK

TeamsFx helps to reduce your tasks by using Microsoft Teams Single-Sign-On (Teams SSO) and accessing cloud resources down to single line statements with zero configuration. You can use TeamsFx SDK in browser and Node.js environment. TeamsFx core functionalities can be accessed in client and server environment. You can write user authentication code in a simplified way for:

* Teams tab
* Teams bot
* Azure Function

## Prerequisites

You need to install the following tools and set up your development environment:

| &nbsp; | Install | For using... |
   | --- | --- | --- |
   | &nbsp; | [Visual Studio Code](https://code.visualstudio.com/download) | JavaScript, TypeScript, or SharePoint Framework (SPFx) build environments. Use version 1.55 or later. |
   | &nbsp; | [Teams Toolkit](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension)| A Microsoft Visual Studio Code extension that creates a project scaffolding for your app. Use 4.0.0 version. |
   | &nbsp; | [Node.js](https://nodejs.org/en/download/) | Back-end JavaScript runtime environment. Use the latest v16 LTS release.|
   | &nbsp; | [Microsoft Teams](https://www.microsoft.com/microsoft-teams/download-app) | Microsoft Teams to collaborate with everyone you work with through apps for chat, meetings, call - all in one place.|
   | &nbsp; | [Microsoft&nbsp;Edge](https://www.microsoft.com/edge) (recommended) or [Google Chrome](https://www.google.com/chrome/) | A browser with developer tools. |

> [!NOTE]
> If your project has installed `botbuilder` related [packages](https://github.com/Microsoft/botbuilder-js#packages) as dependencies, ensure they are of the same version.

You must have working knowledge of:

* [Source code](https://github.com/OfficeDev/TeamsFx/tree/main/packages/sdk)
* [Package (NPM)](https://www.npmjs.com/package/@microsoft/teamsfx)
* [API reference documentation](https://aka.ms/teamsfx-sdk-help)
* [Samples](https://github.com/OfficeDev/TeamsFx-Samples)

## Get started

TeamsFx SDK is pre-configured in the scaffolded project using TeamsFx Toolkit or CLI.
For more information, see [Teams app project](https://github.com/OfficeDev/TeamsFx/blob/main/packages/vscode-extension/README.md).

### Install the `@microsoft/teamsfx` package

Install the TeamsFx SDK for TypeScript or JavaScript with `npm`:

```bash
npm install @microsoft/teamsfx
```

## TeamsFx core functionalities

### TeamsFx class

TeamsFx class instance access all TeamsFx settings from environment variables by default. You can also set customized configuration values to override the default values. Check [override configuration](#override-configuration) for details.
When creating a TeamsFx instance, you also need to specify the identity type.
There are two identity types:

* **User Identity**: Represents the current user of Teams.
* **Application Identity**: Represents the application itself.

    > [!NOTE]
    > For these two identity types, the TeamsFx constructors and methods aren't the same.

You can learn more about user identity and application identity in the following section:

<details>
<summary><b> User identity </b></summary>

| Command | Description |
|----------------|-------------|
| `new TeamsFx(IdentityType.User)`| Application is authenticated as current Teams user. |
| `TeamsFx:setSsoToken()`| User identity in Node.js environment (without browser). |
| `TeamsFx:getUserInfo()` | To get user's basis information. |
| `TeamsFx:login()` | It's used to let user perform consent process, if you want to use SSO to get access token for certain OAuth scopes. |

> [!NOTE]
> You can access resources on behalf of current Teams user.
</details>

<details>
<summary><b> Application identity </b></summary>

| Command | Description |
|----------------|-------------|
| `new TeamsFx(IdentityType.App)`| Application  is authenticated as an application. The permission usually needs administrator's approval.|
| `TeamsFx:getCredential()`| Its provides credential instances automatically corresponding to identity type. |

> [!NOTE]
> You need admin consent for resources.
</details>

### Credential

To initialize TeamsFx, you must choose the required identity type. Post specifying the identity type SDK uses different type of credential class. These represent the identity and get access token by corresponding auth flow. Credential classes implement `TokenCredential` interface that is broadly used in Azure library APIs designed to provide access tokens for specific scopes. Other APIs rely on credential call `TeamsFx:getCredential()` to get an instance of `TokenCredential`. For more information on credential and auth flow related classes, see [credential folder](https://github.com/OfficeDev/TeamsFx/tree/main/packages/sdk/src/credential).

There are three credential classes to simplify authentication. Here's the corresponding scenarios for each credential class target.

<details>
<summary><b> User Identity in browser environment </b></summary>

`TeamsUserCredential` represents Teams current user's identity. For the first time user's credentials are authenticated, then Teams SSO does the On-Behalf-Of flow for token exchange. SDK uses this credential when you choose user identity in browser environment.

The required configurations are: `initiateLoginEndpoint` and `clientId`.
</details>

<details>
<summary><b> User Identity in Node.js environment </b></summary>

`OnBehalfOfUserCredential` uses On-Behalf-Of flow and require Teams SSO token, in Azure function or bot scenarios. TeamsFx SDK uses the following credential when you choose user identity in Node.js environment.

Required configuration: `authorityHost`, `tenantId`, `clientId`, `clientSecret` or `certificateContent`.
</details>

<details>
<summary><b> App Identity in Node.js environment </b></summary>

`AppCredential` represents the app identity. You can use app identity when user isn't involved, for example in a time-triggered automation job. TeamsFx SDK uses the following credential when you choose app identity in Node.js environment.

Required configuration: `tenantId`, `clientId`, `clientSecret` or `certificateContent`.
</details>

### Bot SSO

Bot related classes are stored under [bot folder](https://github.com/OfficeDev/TeamsFx/tree/main/packages/sdk/src/bot).

`TeamsBotSsoPrompt` integrates with bot framework. It simplifies the authentication process when you develop bot application and want to use the bot SSO.

Required configuration: `initiateLoginEndpoint`, `tenantId`, `clientId`, and `applicationIdUri`.

### Supported functions

TeamsFx SDK provides several functions to ease the configuration for third-party libraries. They're located under [core folder](https://github.com/OfficeDev/TeamsFx/tree/main/packages/sdk/src/core).

* Microsoft Graph Service:`createMicrosoftGraphClient` and `MsGraphAuthProvider` help to create authenticated Graph instance.
* SQL:`getTediousConnectionConfig` returns a tedious connection config.

    Required configuration:

  * If you want to use user identity, then `sqlServerEndpoint`, `sqlUsername` and `sqlPassword` are required.
  * If you want to use MSI identity, then `sqlServerEndpoint`and `sqlIdentityId` are required.

### Override configuration

You can pass custom config when creating a new `TeamsFx` instance to override default configuration or set required fields when `environment variables` are missing.

<details>
<summary><b>
For tab project
</b> </summary>

If you've created tab project using Microsoft Visual Studio Code Toolkit, the following config values will be used from pre-configured environment variables:

* authorityHost (REACT_APP_AUTHORITY_HOST)
* tenantId (REACT_APP_TENANT_ID)
* clientId (REACT_APP_CLIENT_ID)
* initiateLoginEndpoint (REACT_APP_START_LOGIN_PAGE_URL)
* applicationIdUri (REACT_APP_START_LOGIN_PAGE_URL)
* apiEndpoint (REACT_APP_FUNC_ENDPOINT) // only used when there's a backend function
* apiName (REACT_APP_FUNC_NAME) // only used when there's a backend function

</details>

<details>
<summary><b>
For Azure function or bot project
</b></summary>

If you've created Azure function or bot project using Visual Studio Code Toolkit, the following config values will be used from pre-configured environment variables:

* initiateLoginEndpoint (INITIATE_LOGIN_ENDPOINT)
* authorityHost (M365_AUTHORITY_HOST)
* tenantId (M365_TENANT_ID)
* clientId (M365_CLIENT_ID)
* clientSecret (M365_CLIENT_SECRET)
* applicationIdUri (M365_APPLICATION_ID_URI)
* apiEndpoint (API_ENDPOINT)
* sqlServerEndpoint (SQL_ENDPOINT) // only used when there's an sql instance
* sqlUsername (SQL_USER_NAME) // only used when there's an sql instance
* sqlPassword (SQL_PASSWORD) // only used when there's an sql instance
* sqlDatabaseName (SQL_DATABASE_NAME) // only used when there's an sql instance
* sqlIdentityId (IDENTITY_ID) // only used when there's an sql instance

</details>

### Error handling

Basic type of API error response is `ErrorWithCode`, which contains error code and error message. For example, to filter out specific error, you can use the following snippet:

```typescript
try {
  const teamsfx = new TeamsFx();
  await teamsfx.login("User.Read");
} catch (err: unknown) {
  if (err instanceof ErrorWithCode && err.code !== ErrorCode.ConsentFailed) {
    throw err;
  } else {
    // Silently fail because user cancels the consent dialog
    return;
  }
}
```

If credential instance is used in other library such as Microsoft Graph, it's possible that error is caught and transformed.

## Microsoft Graph Scenarios

This section provides several code snippets for common scenarios that are related to Microsoft Graph. In such scenarios, user can call APIs using different permissions in different ends(frontend/backend).

* User delegate permission in frontend (Use TeamsUserCredential)
    <details>
    <summary><b>Use graph API in tab app</b></summary>

    This code snippet shows you how to use `TeamsFx` and `createMicrosoftGraphClient` to get user profiles from Microsoft Graph. It also shows you how to catch and resolve a `GraphError`.

    1. Import the classes needed.

    ```typescript
    import {
      createMicrosoftGraphClient,
      TeamsFx,
    } from "@microsoft/teamsfx";
    ```

    2. Use `TeamsFx.login()` to get user consent.

    ```typescript
    // Put these code in a call-to-action callback function to avoid browser blocking automatically showing up pop-ups.
    await teamsfx.login(["User.Read"]); // Login with scope
    ```

    3. You can initialize a TeamsFx instance and graph client and get information from MS Graph by this client.

    ```typescript
    try {
      const teamsfx = new TeamsFx();
      const graphClient = createMicrosoftGraphClient(teamsfx, ["User.Read"]); // Initializes MS Graph SDK using our MsGraphAuthProvider
      const profile = await graphClient.api("/me").get();
    } catch (err: unknown) {
      // ErrorWithCode is handled by Graph client
      if (err instanceof GraphError && err.code?.includes(ErrorCode.UiRequiredError)) {
        // Need to show login button to ask for user consent.
      }
    }
    ```

    For more information on sample to use Graph API in tab app, see the [hello-world-tab sample](https://github.com/OfficeDev/TeamsFx-Samples/tree/dev/hello-world-tab).

    </details>

    <details>
    <summary><b>Integration with Microsoft Graph Toolkit</b></summary>

    The [Microsoft Graph Toolkit (mgt)](https://aka.ms/mgt) library is a collection of various authentication providers and UI components powered by Microsoft Graph.

    The `@microsoft/mgt-teamsfx-provider` package exposes the `TeamsFxProvider` class that uses `TeamsFx` class to sign in users and acquire tokens to use with Graph.

    1. You can install the following required packages:

    ```bash
    npm install @microsoft/mgt-element @microsoft/mgt-teamsfx-provider @microsoft/teamsfx
    ```

    2. Initialize the provider inside your component.

    ```typescript
     // Import the providers and credential at the top of the page
    import {Providers} from '@microsoft/mgt-element';
    import {TeamsFxProvider} from '@microsoft/mgt-teamsfx-provider';
    import {TeamsUserCredential} from "@microsoft/teamsfx";

    const scope = ["User.Read"];
    const teamsfx = new TeamsFx();
    const provider = new TeamsFxProvider(teamsfx, scope);
    Providers.globalProvider = provider;   
    ```

    3. You can use the `teamsfx.login(scopes)` method to get required access token.

    ```typescript
    // Put these code in a call-to-action callback function to avoid browser blocking automatically showing up pop-ups. 
    await teamsfx.login(this.scope);
    Providers.globalProvider.setState(ProviderState.SignedIn);
    ```

    4. You can now add any component in your HTML page or in your `render()` method with React to use the `TeamsFx` context to access Microsoft Graph.

    ```html
    <mgt-person query="me" view="threeLines"></mgt-person>
    ```

    ```typescript
    public render(): void {
    return (
        <div>
            <Person personQuery="me" view={PersonViewType.threelines}></Person>
        </div>
    );
    }    
    ```

    For more information on sample to initialize the TeamsFx provider, see the [Contacts Exporter sample](https://github.com/OfficeDev/TeamsFx-Samples/tree/dev/graph-toolkit-contact-exporter).

    </details>

* User delegate permission in backend (Use OnBehalfOfUserCredential)
    <details>
    <summary><b>Use Graph API in bot Application</b></summary>

    This code snippet shows you how to use `TeamsBotSsoPrompt` to set a dialog and then sign-in to get an access token.

    1. Initialize and add `TeamsBotSsoPrompt` to dialog set.

    ```typescript
    const { ConversationState, MemoryStorage } = require("botbuilder");
    const { DialogSet, WaterfallDialog } = require("botbuilder-dialogs");
    const { TeamsBotSsoPrompt } = require("@microsoft/teamsfx");

    const convoState = new ConversationState(new MemoryStorage());
    const dialogState = convoState.createProperty("dialogState");
    const dialogs = new DialogSet(dialogState);

    const teamsfx = new TeamsFx();
    dialogs.add(
      new TeamsBotSsoPrompt(teamsfx, "TeamsBotSsoPrompt", {
        scopes: ["User.Read"],
      })
    );    
    ```

    2. Begin the dialog and sign-in.

    ```typescript
    dialogs.add(
      new WaterfallDialog("taskNeedingLogin", [
        async (step) => {
          return await step.beginDialog("TeamsBotSsoPrompt");
        },
        async (step) => {
          const token = step.result;
          if (token) {
            // ... continue with task needing access token ...
          } else {
            await step.context.sendActivity(`Sorry... We couldn't log you in. Try again later.`);
            return await step.endDialog();
          }
        },
      ])
    );    
    ```

    For more information on sample to use graph API in bot application, see [bot-sso sample](https://github.com/OfficeDev/TeamsFx-Samples/tree/dev/bot-sso).

    </details>

    <details>
    <summary><b>Use Graph API in Message Extension</b></summary>

    This code snippet shows you how to override `handleTeamsMessagingExtensionQuery` extends from `TeamsActivityHandler`, and use `handleMessageExtensionQueryWithToken` provided by TeamsFx sdk to sign-in to get an access token:

    ```typescript
    public async handleTeamsMessagingExtensionQuery(context: TurnContext, query: any): Promise<any> {
      return await handleMessageExtensionQueryWithToken(context, null, 'User.Read', 
        async (token: MessageExtensionTokenResponse) => {
          // ... continue to query with access token ...
        });
    }    
    ```

    For more information on sample to use graph API in message extension, see [message-extension-sso-sample](https://aka.ms/teamsfx-me-sso-sample).
    </details>

    <details>
    <summary><b>Use Graph API in Command Bot</b></summary>

    This code snippet shows you how to implement `TeamsFxBotSsoCommandHandler` for command bot to call Microsoft API.

    ```typescript
    import { Activity, TurnContext } from "botbuilder";
    import {
      CommandMessage,
      TriggerPatterns,
      TeamsFx,
      createMicrosoftGraphClient,
      TeamsFxBotSsoCommandHandler,
      TeamsBotSsoPromptTokenResponse,
    } from "@microsoft/teamsfx";

    export class ProfileSsoCommandHandler implements TeamsFxBotSsoCommandHandler {
      triggerPatterns: TriggerPatterns = "profile";

      async handleCommandReceived(
        context: TurnContext,
        message: CommandMessage,
        tokenResponse: TeamsBotSsoPromptTokenResponse,
      ): Promise<string | Partial<Activity> | void> {
        // Init TeamsFx instance with SSO token
        const teamsfx = new TeamsFx().setSsoToken(tokenResponse.ssoToken);

        // Add scope for your Azure AD app. For example: Mail.Read, etc.
        const graphClient = createMicrosoftGraphClient(teamsfx, ["User.Read"]);
      
        // Call graph api use `graph` instance to get user profile information
        const me = await graphClient.api("/me").get();

        if (me) {
          // Bot will send the user profile info to user
          return `Your command is '${message.text}' and you're logged in as ${me.displayName}`;
        } else {
          return "Could not retrieve profile information from Microsoft Graph.";
        }
      }
    }    
    ```

    For more information about how to use this class in command bot, see [Add single sign-on to Teams app](add-single-sign-on.md). And there's also a [command-bot-with-sso](https://github.com/OfficeDev/TeamsFx-Samples/tree/dev/command-bot-with-sso) sample project, which you can try sso command bot.

    </details>

    <details>
    <summary><b>Call Azure Function in tab app: On-Behalf-Of flow</b></summary>

    This code snippet shows you how to use `CreateApiClient` or `axios` library to call Azure Function, and how to call Graph API in Azure function to get user profiles.

    1. You can use `CreateApiClient` provided by TeamsFx sdk to call Azure Function:

    ```typescript
    async function callFunction(teamsfx?: TeamsFx) {
      const teamsfx = new TeamsFx();

      // Get the credential.
      const credential = teamsfx.getCredential(); 
      // Create an API client by providing the token and endpoint.
      const apiClient = CreateApiClient(
        teamsfx.getConfig("YOUR_API_ENDPOINT"), // Create an API Client that uses SSO token to authenticate requests
        new BearerTokenAuthProvider(async () =>  (await credential.getToken(""))!.token) // Call API hosted in Azure Functions on behalf of user to inject token to request header
      );

      // Send a GET request to "RELATIVE_API_PATH", "/api/functionName" for example.
      const response = await apiClient.get("RELATIVE_API_PATH");
      return response.data;
    }    
    ```

    You can also use `axios` library to call Azure Function.

    ```typescript
    async function callFunction(teamsfx?: TeamsFx) {
      const accessToken = await teamsfx.getCredential().getToken(""); // Get SSO token 
      // teamsfx.getConfig("apiEndpoint") will read REACT_APP_FUNC_ENDPOINT environment variable 
      const endpoint = teamsfx.getConfig("apiEndpoint");
      const response = await axios.default.get(endpoint + "/api/" + functionName, {
        headers: {
          authorization: "Bearer " + accessToken.token,
        },
      });
      return response.data;
    }    
    ```

    2. Call Graph API in Azure function on-behalf of user in response.

    ```typescript
    export default async function run(
      context: Context,
      req: HttpRequest,
      teamsfxContext: TeamsfxContext
    ): Promise<Response> {
      const res: Response = { status: 200, body: {},};
      // ...
      teamsfx = new TeamsFx().setSsoToken(accessToken);
      // Query user's information from the access token.
      try {
        const currentUser: UserInfo = await teamsfx.getUserInfo();
        if (currentUser && currentUser.displayName) {
          res.body.userInfoMessage = `User display name is ${currentUser.displayName}.`;
        } else {
          res.body.userInfoMessage = "No user information was found in access token.";
        }
      } catch (e) {
      }
      // Create a graph client to access user's Microsoft 365 data after user has consented.
      try {
        const graphClient: Client = createMicrosoftGraphClient(teamsfx, [".default"]);
        const profile: any = await graphClient.api("/me").get();
        res.body.graphClientMessage = profile;
      } catch (e) {
      }
      return res;
    }    
    ```

    For more information on sample to use graph API in bot application, see  [hello-world-tab-with-backend sample](https://github.com/OfficeDev/TeamsFx-Samples/tree/dev/hello-world-tab-with-backend).

    </details>

* Application permission in backend
    <details>
    <summary><b>Use certificate-based authentication in Azure Function</b></summary>

    This code snippet shows you how to use certificate-based application permission to get the token that can be used to call Graph API.

    1. You can initialize the `authConfig` by providing a `PEM-encoded key certificate`.

    ```typescript
    const authConfig = {
      clientId: process.env.M365_CLIENT_ID,
      certificateContent: "The content of a PEM-encoded public/private key certificate",
      authorityHost: process.env.M365_AUTHORITY_HOST,
      tenantId: process.env.M365_TENANT_ID,
    };    
    ```

    2. You can use the `authConfig` to get the token.

    ```typescript
    const teamsfx = new TeamsFx(IdentityType.App);
    teamsfx.setCustomeConfig(authConfig);
    const token = teamsfx.getCredential().getToken();    
    ```

    </details>

    <details>
    <summary><b>Use client secret authentication in Azure Function</b></summary>

    This code snippet shows you how to use client secret application permission to get the token that can be used to call Graph API.

    1. You can initialize the `authConfig` by providing a `client secret`.

    ```typescript
    const authConfig = {
      clientId: process.env.M365_CLIENT_ID,
      clientSecret: process.env.M365_CLIENT_SECRET,
      authorityHost: process.env.M365_AUTHORITY_HOST,
      tenantId: process.env.M365_TENANT_ID,
    };    
    ```

    2. You can use the `authConfig` to get the token.

    ```typescript
    const teamsfx = new TeamsFx(IdentityType.App);
    teamsfx.setCustomeConfig(authConfig);
    const token = teamsfx.getCredential().getToken();    
    ```

    For more information on sample to use graph API in bot application, see the [hello-world-tab-with-backend sample](https://github.com/OfficeDev/TeamsFx-Samples/tree/dev/hello-world-tab-with-backend).

    </details>

## Other scenarios

This section provides several code snippets for other scenarios that are related to Microsoft Graph. You can create API client in Bot or Azure Function and access SQL database in Azure Function.

  <details>
  <summary><b>Create API client to call existing API in Bot or Azure Function</b></summary>

  This code snippet shows you how to call an existing API in Bot by `ApiKeyProvider`.

  ```typescript
  const teamsfx = new TeamsFx();

  // Create an API Key auth provider. In addition to APiKeyProvider, following auth providers are also available:
  // BearerTokenAuthProvider, BasicAuthProvider, CertificateAuthProvider.
  const authProvider = new ApiKeyProvider("YOUR_API_KEY_NAME",
    teamsfx.getConfig("YOUR_API_KEY_VALUE"), // This reads the value of YOUR_API_KEY_VALUE environment variable.
    ApiKeyLocation.Header
  );

  // Create an API client using above auth provider.
  // You can also implement AuthProvider interface and use it here.
  const apiClient = createApiClient(
    teamsfx.getConfig("YOUR_API_ENDPOINT"), // This reads YOUR_API_ENDPOINT environment variable.
    authProvider
  );

  // Send a GET request to "RELATIVE_API_PATH", "/api/apiname" for example.
  const response = await apiClient.get("RELATIVE_API_PATH");  
  ```

  </details>

  <details>
  <summary><b>Access SQL database in Azure Function</b></summary>

  Use `tedious` library to access SQL and use `DefaultTediousConnectionConfiguration` that manages authentication. You can also compose connection config of other SQL libraries based on the result of `sqlConnectionConfig.getConfig()`.

  1. Set the connection configuration.

  ```typescript
  // Equivalent to:
  // const sqlConnectConfig = new DefaultTediousConnectionConfiguration({
  //    sqlServerEndpoint: process.env.SQL_ENDPOINT,
  //    sqlUsername: process.env.SQL_USER_NAME,
  //    sqlPassword: process.env.SQL_PASSWORD,
  // });
  const teamsfx = new TeamsFx();
  // If there's only one SQL database
  const config = await getTediousConnectionConfig(teamsfx);
  // If there are multiple SQL databases
  const config2 = await getTediousConnectionConfig(teamsfx, "your database name");  
  ```

  2. Connect to your database.

  ```typescript
  const connection = new Connection(config);
  connection.on("connect", (error) => {
    if (error) {
      console.log(error);
    }
  });  
  ```

  > [!NOTE]
  > For more information on sample to access SQL database in Azure function, see [share-now sample](https://github.com/OfficeDev/TeamsFx-Samples/tree/dev/share-now).

</details>

## Advanced Customization

### Configure log

You can set customer log level and redirect outputs when using this library.

> [!NOTE]
> Logging is turned off by default, you can turn it on by setting log level.

#### Enable log by setting log level

When you set log level then Logging gets enabled. It prints log information to console by default.

Set log level using the following snippet:

```typescript
// Only need the warning and error messages.
setLogLevel(LogLevel.Warn);
```

> [!NOTE]
> You can redirect log output by setting custom logger or log function.

#### Redirect by setting custom logger

```typescript
setLogLevel(LogLevel.Info);
// Set another logger if you want to redirect to Application Insights in Azure Function
setLogger(context.log);
```

#### Redirect by setting custom log function

```typescript
setLogLevel(LogLevel.Info);
// Only log error message to Application Insights in bot application.
setLogFunction((level: LogLevel, message: string) => {
  if (level === LogLevel.Error) {
    this.telemetryClient.trackTrace({
      message: message,
      severityLevel: Severity.Error,
    });
  }
});
```

> [!NOTE]
> Log functions don't take effect if you set a custom logger.

## Upgrade latest SDK version

If you're using the version of SDK that has `loadConfiguration()`, you can follow these steps to upgrade to the latest SDK version:

1. Remove `loadConfiguration()` and pass customized settings using `new TeamsFx(IdentityType.User, { ...customConfig })`
2. Replace `new TeamsUserCredential()` with `new TeamsFx()`.
3. Replace `new M365TenantCredential()` with `new TeamsFx(IdentityType.App)`.
4. Replace `new OnBehalfOfUserCredential(ssoToken)` with `new TeamsFx().setSsoToken(ssoToken)`.
5. Pass the instance of `TeamsFx` to helper functions to replace credential instance.

## Next step

For detailed examples on how to use TeamsFx SDK [Samples](https://github.com/OfficeDev/TeamsFx-Samples) project.

## See also

[Microsoft TeamsFx sample gallery](https://github.com/OfficeDev/TeamsFx-Samples).
