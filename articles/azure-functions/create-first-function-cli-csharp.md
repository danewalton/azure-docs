---
title: Create a C# function from the command line - Azure Functions
description: Learn how to create a C# function from the command line, then publish the local project to serverless hosting in Azure Functions.
ms.date: 09/14/2021
ms.topic: quickstart
ms.custom: [devx-track-csharp, devx-track-azurecli, devx-track-azurepowershell]
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./create-first-function-cli-csharp-ieux
zone_pivot_groups: runtime-version-programming-functions
---

# Quickstart: Create a C# function in Azure from the command line

[!INCLUDE [functions-runtime-version-dotnet](../../includes/functions-runtime-version-dotnet.md)]

In this article, you use command-line tools to create a C# function that responds to HTTP requests. After testing the code locally, you deploy it to the serverless environment of Azure Functions.

[!INCLUDE [functions-dotnet-execution-model](../../includes/functions-dotnet-execution-model.md)]

Completing this quickstart incurs a small cost of a few USD cents or less in your Azure account.

There is also a [Visual Studio Code-based version](create-first-function-vs-code-csharp.md) of this article.

## Configure your local environment

Before you begin, you must have the following:

::: zone pivot="programming-runtime-functions-v3"
[!INCLUDE [functions-cli-dotnet-prerequisites](../../includes/functions-cli-dotnet-prerequisites.md)]
::: zone-end
::: zone pivot="programming-runtime-functions-v4"
# [In-process](#tab/in-process)

+ [.NET 6.0 SDK](https://dotnet.microsoft.com/download)

+ [Azure Functions Core Tools](./functions-run-local.md#v2) version 4.x.

+ One of the following tools for creating Azure resources:

    + [Azure CLI](/cli/azure/install-azure-cli) version 2.4 or later.

    + The Azure [Az PowerShell module](/powershell/azure/install-az-ps) version 5.9.0 or later.

# [Isolated process](#tab/isolated-process)

+ [.NET 6.0 SDK](https://dotnet.microsoft.com/download/dotnet/6.0)

+ [.NET Core 3.1 SDK](https://dotnet.microsoft.com/download). Required by the build process.

+ [Azure Functions Core Tools](./functions-run-local.md#v2) version 4.x.

+ One of the following tools for creating Azure resources:

    + [Azure CLI](/cli/azure/install-azure-cli) version 2.4 or later.

    + The Azure [Az PowerShell module](/powershell/azure/install-az-ps) version 5.9.0 or later.
---
::: zone-end

You also need an Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

### Prerequisite check

Verify your prerequisites, which depend on whether you are using Azure CLI or Azure PowerShell for creating Azure resources:

# [Azure CLI](#tab/azure-cli)

+ In a terminal or command window, run `func --version` to check that the Azure Functions Core Tools are version 3.x.

+ Run `dotnet --list-sdks` to check that the required versions are installed.

+ Run `az --version` to check that the Azure CLI version is 2.4 or later.

+ Run `az login` to sign in to Azure and verify an active subscription.

# [Azure PowerShell](#tab/azure-powershell)

+ In a terminal or command window, run `func --version` to check that the Azure Functions Core Tools are version 3.x.

+ Run `dotnet --list-sdks` to check that the required versions are installed.

+ Run `(Get-Module -ListAvailable Az).Version` and verify version 5.0 or later.

+ Run `Connect-AzAccount` to sign in to Azure and verify an active subscription.

---

## Create a local function project

In Azure Functions, a function project is a container for one or more individual functions that each responds to a specific trigger. All functions in a project share the same local and hosting configurations. In this section, you create a function project that contains a single function.

1. Run the `func init` command, as follows, to create a functions project in a folder named *LocalFunctionProj* with the specified runtime:

    # [In-process](#tab/in-process)

    ```console
    func init LocalFunctionProj --dotnet
    ```

    # [Isolated process](#tab/isolated-process)

    ```console
    func init LocalFunctionProj --worker-runtime dotnet-isolated
    ```
    ---

1. Navigate into the project folder:

    ```console
    cd LocalFunctionProj
    ```

    This folder contains various files for the project, including configurations files named [local.settings.json](functions-develop-local.md#local-settings-file) and [host.json](functions-host-json.md). Because *local.settings.json* can contain secrets downloaded from Azure, the file is excluded from source control by default in the *.gitignore* file.

1. Add a function to your project by using the following command, where the `--name` argument is the unique name of your function (HttpExample) and the `--template` argument specifies the function's trigger (HTTP).

    ```console
    func new --name HttpExample --template "HTTP trigger" --authlevel "anonymous"
    ```

    `func new` creates a HttpExample.cs code file.

### (Optional) Examine the file contents

If desired, you can skip to [Run the function locally](#run-the-function-locally) and examine the file contents later.

#### HttpExample.cs

The function code generated from the template depends on the type of compiled C# project.

# [In-process](#tab/in-process)

*HttpExample.cs* contains a `Run` method that receives request data in the `req` variable is an [HttpRequest](/dotnet/api/microsoft.aspnetcore.http.httprequest) that's decorated with the **HttpTriggerAttribute**, which defines the trigger behavior.

:::code language="csharp" source="~/functions-docs-csharp/http-trigger-template/HttpExample.cs":::

The return object is an [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) that returns a response message as either an [OkObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult) (200) or a [BadRequestObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.badrequestobjectresult) (400).

# [Isolated process](#tab/isolated-process)

*HttpExample.cs* contains a `Run` method that receives request data in the `req` variable is an [HttpRequestData](/dotnet/api/microsoft.azure.functions.worker.http.httprequestdata) object that's decorated with the **HttpTriggerAttribute**, which defines the trigger behavior. Because of the isolated process model,    `HttpRequestData` is a representation of the actual `HttpRequest`, and not the request object itself.

:::code language="csharp" source="~/functions-docs-csharp/http-trigger-isolated/HttpExample.cs":::

The return object is an [HttpResponseData](/dotnet/api/microsoft.azure.functions.worker.http.httpresponsedata) object that contains the data that's handed back to the HTTP response.

---

To learn more, see [Azure Functions HTTP triggers and bindings](./functions-bindings-http-webhook.md?tabs=csharp).

## Run the function locally

1. Run your function by starting the local Azure Functions runtime host from the *LocalFunctionProj* folder:

    ```
    func start
    ```

    Toward the end of the output, the following lines should appear:

    <pre>
    ...

    Now listening on: http://0.0.0.0:7071
    Application started. Press Ctrl+C to shut down.

    Http Functions:

            HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
    ...

    </pre>

    >[!NOTE]
    > If HttpExample doesn't appear as shown above, you likely started the host from outside the root folder of the project. In that case, use **Ctrl**+**C** to stop the host, navigate to the project's root folder, and run the previous command again.

1. Copy the URL of your `HttpExample` function from this output to a browser:

    # [In-process](#tab/in-process)

     To the function URL, append the query string `?name=<YOUR_NAME>`, making the full URL like `http://localhost:7071/api/HttpExample?name=Functions`. The browser should display a response message that echoes back your query string value. The terminal in which you started your project also shows log output as you make requests.

    # [Isolated process](#tab/isolated-process)

    Browse to the function URL and you should receive a _Welcome to Azure Functions_ message.

    ---

1. When you're done, use **Ctrl**+**C** and choose `y` to stop the functions host.

[!INCLUDE [functions-create-azure-resources-cli](../../includes/functions-create-azure-resources-cli.md)]

4. Create the function app in Azure:

    # [Azure CLI](#tab/azure-cli/in-process)

    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location <REGION> --runtime dotnet --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```
    The [az functionapp create](/cli/azure/functionapp#az_functionapp_create) command creates the function app in Azure.

    # [Azure CLI](#tab/azure-cli/isolated-process)

    ```azurecli
    az functionapp create --resource-group AzureFunctionsQuickstart-rg --consumption-plan-location <REGION> --runtime dotnet-isolated --functions-version 3 --name <APP_NAME> --storage-account <STORAGE_NAME>
    ```

    The [az functionapp create](/cli/azure/functionapp#az_functionapp_create) command creates the function app in Azure.

    # [Azure PowerShell](#tab/azure-powershell/in-process)

    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -Runtime dotnet -FunctionsVersion 3 -Location '<REGION>'
    ```

    The [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp) cmdlet creates the function app in Azure.

    # [Azure PowerShell](#tab/azure-powershell/isolated-process)

    ```azurepowershell
    New-AzFunctionApp -Name <APP_NAME> -ResourceGroupName AzureFunctionsQuickstart-rg -StorageAccount <STORAGE_NAME> -Runtime dotnet-isolated -FunctionsVersion 3 -Location '<REGION>'
    ```

    The [New-AzFunctionApp](/powershell/module/az.functions/new-azfunctionapp) cmdlet creates the function app in Azure.

    ---

    ::: zone pivot="programming-runtime-functions-v4"
    > [!NOTE]
    > This command creates a function app using the 3.x version of the Azure Functions runtime. The `func azure functionapp publish` command that you'll run later updates the app to version 4.x.
    ::: zone-end

    In the previous example, replace `<STORAGE_NAME>` with the name of the account you used in the previous step, and replace `<APP_NAME>` with a globally unique name appropriate to you. The `<APP_NAME>` is also the default DNS domain for the function app.

    This command creates a function app running in your specified language runtime under the [Azure Functions Consumption Plan](consumption-plan.md), which is free for the amount of usage you incur here. The command also provisions an associated Azure Application Insights instance in the same resource group, with which you can monitor your function app and view logs. For more information, see [Monitor Azure Functions](functions-monitoring.md). The instance incurs no costs until you activate it.

[!INCLUDE [functions-publish-project-cli](../../includes/functions-publish-project-cli.md)]

## Invoke the function on Azure

Because your function uses an HTTP trigger and supports GET requests, you invoke it by making an HTTP request to its URL. It's easiest to do this in a browser.

# [In-process](#tab/in-process)

Copy the complete **Invoke URL** shown in the output of the publish command into a browser address bar, appending the query parameter `?name=Functions`. When you navigate to this URL, the browser should display similar output as when you ran the function locally.

# [Isolated process](#tab/isolated-process)

Copy the complete **Invoke URL** shown in the output of the publish command into a browser address bar. When you navigate to this URL, the browser should display similar output as when you ran the function locally.

---

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

[!INCLUDE [functions-cleanup-resources-cli](../../includes/functions-cleanup-resources-cli.md)]

## Next steps

# [In-process](#tab/in-process)

> [!div class="nextstepaction"]
> [Connect to Azure Queue Storage](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-csharp&tabs=in-process)

# [Isolated process](#tab/isolated-process)

> [!div class="nextstepaction"]
> [Connect to Azure Queue Storage](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-csharp&tabs=isolated-process)

---
