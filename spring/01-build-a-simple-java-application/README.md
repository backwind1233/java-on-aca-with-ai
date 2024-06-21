# 01 - Build a simple Java application

__This guide is part of the [Build, Run and Monitor Intelligent Java Apps on Azure Container Apps and Azure OpenAI](../README.md)__

In this section, we'll build a simple Java application and deploy it to Azure Container Apps. This will give us a starting point for adding advanced technologies in later sections.

---

## Create a simple Java application

The application that we create in this guide is [available here](simple-application/).

A typical way to create Java applications is to use the Spring Initializr at [https://start.spring.io/](https://start.spring.io/). Feel free to explore it outside this training. **For the purposes of this training, we will only invoke the Spring Initializr site via the `curl` command**.

>💡 __Note:__ All subsequent commands in this workshop should be run from the same directory, except where otherwise indicated via `cd` commands.

In an __empty__ directory execute the `curl` command line below:

```bash
curl https://start.spring.io/starter.tgz \
    -d type=maven-project \
    -d dependencies=web \
    -d baseDir=simple-application \
    -d name=simple-application \
    -d bootVersion=3.2.5 \
    -d javaVersion=17 \
    | tar -xzvf -
```

> We force the Spring Boot version to be 3.2.5, and keep default settings that use the `com.example.demo` package.

## Add a new Spring MVC Controller

In the `simple-application/src/main/java/com/example/demo` directory, create a
new file called `HelloController.java` next to `SimpleApplication.java` file with
the following content:

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from Java application on Azure Container Apps\n";
    }
}
```

The final project is available in the ["simple-application" folder](simple-application/).

## Test the project locally

Run the project:

```bash
cd simple-application
./mvnw clean package -DskipTests
java -jar target/demo-0.0.1-SNAPSHOT.jar &
cd ..
```

Requesting the `/hello` endpoint should return the "Hello from Java application on Azure Container Apps" message.

```bash
curl http://127.0.0.1:8080/hello
```

Finally, kill running app:

```bash
kill %1
```

## Build and deploy the application on Azure Container Apps

This section shows how to create a container app and then deploy your application to it.

### Setup

To sign in to Azure from the CLI, run the following command and follow the prompts to complete the authentication process.

```bash
az login
```

Ensure you're running the latest version of the CLI via the upgrade command.

```bash
az upgrade
```

Next, install, or update the Azure Container Apps extension for the CLI.

```bash
az extension add --name containerapp --upgrade
```

Select the Azure Subscription you want to use.

```bash
az account set --subscription "YOUR_SUBSCRIPTION_ID"
```

> 💡 If necessary, you can query for your subscription ID.
> 
> ```bash
> az account list --output table
> ```

Register the `Microsoft.App` and `Microsoft.OperationalInsights` namespaces if they're not registered in your Azure subscription.

```bash
az provider register --namespace Microsoft.App
az provider register --namespace Microsoft.OperationalInsights
az provider register --namespace Microsoft.ServiceLinker
```

Now your Azure CLI setup is complete, you can define the environment variables that are used throughout the following guide.

```bash
RESOURCE_GROUP="java-on-aca"
LOCATION="canadacentral"
ENVIRONMENT="java-on-aca-env"
APP_NAME="simple-app"
```

### Deploy the application

Build and deploy your first container app from your local JAR file with the `containerapp up` command.

This command:

- Builds the container image
- Creates the Container Apps environment with a Log Analytics workspace
- Creates and deploys the container app using the built container image

```bash
cd simple-application
./mvnw clean package -DskipTests
az containerapp up \
    --name $APP_NAME \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --environment $ENVIRONMENT \
    --artifact ./target/demo-0.0.1-SNAPSHOT.jar \
    --ingress external \
    --target-port 8080
cd ..
```

The `containerapp up` command will output detail build and deploy steps performed.

## Test the project in the cloud

Go to [the Azure portal](https://portal.azure.com):

- Look for your container app named as `$APP_NAME` in your resource group named as `$RESOURCE_GROUP`
- Find the "Application Url" in the "Essentials" section
![Application Url](media/01-application-url.png)
- This will give you something like:
  `https://simple-app.bluedune-c2667fb6.canadacentral.azurecontainerapps.io`
- Append `/hello` to the Url.  Failure to do this will result in a "404 not found"

You can now `curl` again to test the `/hello` endpoint, this time served by Azure Container Apps. For example.

```bash
curl https://simple-app.bluedune-c2667fb6.canadacentral.azurecontainerapps.io/hello
```

Alternatively, you can find the "Application Url" in CLI command.

```bash
az containerapp show \
    --name $APP_NAME \
    --resource-group $RESOURCE_GROUP \
    --query properties.configuration.ingress.fqdn
```

## Conclusion

Congratulations, you have deployed your first Java application to Azure Container Apps!

If you need to check your code, the final project is available in the ["simple-application" folder](simple-application/).

Here is the final script to build and deploy everything that was done in this guide:

```
curl https://start.spring.io/starter.tgz \
    -d type=maven-project \
    -d dependencies=web \
    -d baseDir=simple-application \
    -d name=simple-application \
    -d bootVersion=3.2.5 \
    -d javaVersion=17 \
    | tar -xzvf -
cd simple-application
cat > HelloController.java << EOF
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from Java application on Azure Container Apps";
    }
}
EOF
mv HelloController.java src/main/java/com/example/demo/HelloController.java
az containerapp up \
    --name $APP_NAME \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --environment $ENVIRONMENT \
    --artifact ./target/demo-0.0.1-SNAPSHOT.jar \
    --ingress external \
    --target-port 8080
cd ..
```

---

⬅️ Previous guide: [00 - Setup your environment](../00-setup-your-environment/README.md)

➡️ Next guide: [02 - Create Managed Eureka Server for Spring](../02-create-managed-eureka-server-for-spring/README.md)
