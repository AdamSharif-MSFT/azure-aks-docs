---
title: "Deploy WebLogic Server on Azure Kubernetes Service using the Azure portal"
description: Shows how to quickly stand up WebLogic Server on Azure Kubernetes Service.
author: KarlErickson
ms.author: edburns
ms.topic: how-to
ms.date: 06/22/2023
ms.custom: devx-track-java, devx-track-javaee, devx-track-javaee-wls, devx-track-javaee-wls-aks, devx-track-extended-java
---

# Deploy a Java application with WebLogic Server on an Azure Kubernetes Service (AKS) cluster

This article demonstrates how to:

- Run your Java, Java EE, or Jakarta EE on Oracle WebLogic Server (WLS).
- Stand up a WLS cluster using the Azure Marketplace offer.
- Build the application Docker image to serve as auxiliary image to provide WebLogic Deploy Tooling (WDT) models and applications.
- Deploy the containerized application to the existing WLS cluster on AKS.

This article uses the Azure Marketplace offer for WLS to accelerate your journey to AKS. The offer automatically provisions a number of Azure resources including an Azure Container Registry instance, an AKS cluster, an Azure App Gateway Ingress Controller (AGIC) instance, the WebLogic Operator, a container image including WebLogic runtime and a WLS cluster without application. Then, this article introduces building an auxiliary image step by step to update an existing WLS cluster. The auxiliary image is to provide application and WDT models.

For full automation, you can select your appliation and configure datasource connetion from Azure portal before the offer deployment. To see the offer, visit the [Azure portal](https://aka.ms/wlsaks). 

For step-by-step guidance in setting up WebLogic Server on Azure Kubernetes Service, see the official documentation from Oracle at [Azure Kubernetes Service](https://oracle.github.io/weblogic-kubernetes-operator/samples/azure-kubernetes-service/).

## Prerequisites

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
- Ensure the Azure identity you use to sign in and complete this article has either the [Owner](/azure/role-based-access-control/built-in-roles#owner) role in the current subscription or the [Contributor](/azure/role-based-access-control/built-in-roles#contributor) and [User Access Administrator](/azure/role-based-access-control/built-in-roles#user-access-administrator) roles in the current subscription. For an overview of Azure roles, see [What is Azure role-based access control (Azure RBAC)?](/azure/role-based-access-control/overview) For details on the specific roles required by WLS on AKS, see [Azure built-in roles](/azure/role-based-access-control/built-in-roles).
- Have the credentials for an Oracle single sign-on (SSO) account. To create one, see [Create Your Oracle Account](https://aka.ms/wls-aks-create-sso-account).
- Accept the license terms for WLS.
  - Visit the [Oracle Container Registry](https://container-registry.oracle.com/) and sign in.
  - If you have a support entitlement, select **Middleware**, then search for and select **weblogic_cpu**.
  - If you don't have a support entitlement from Oracle, select **Middleware**, then search for and select **weblogic**.
    > [!NOTE]
    > Get a support entitlement from Oracle before going to production. Failure to do so results in running insecure images that are not patched for critical security flaws. For more information on Oracle's critical patch updates, see [Critical Patch Updates, Security Alerts and Bulletins](https://www.oracle.com/security-alerts/) from Oracle.
  - Accept the license agreement.
- Prepare a local machine with Unix-like operating system installed (for example, Ubuntu, Azure Linux, macOS, Windows Subsystem for Linux). 
  - [Azure CLI](/cli/azure); use `az --version` to test if az works. This document was tested with version 2.55.1.
  - [Docker](https://docs.docker.com/get-docker). This document was tested with Docker version 20.10.7. Use `docker info` to test if Docker Daemon is running.
  - [kubectl](https://kubernetes-io-vnext-staging.netlify.com/docs/tasks/tools/install-kubectl/); use `kubectl version` to test if kubectl works. This document was tested with version v1.21.2.
  - A Java JDK compatible with the version of WLS you intend to run. The article directs you to install a version of WLS that uses JDK 11. Azure recommends [Microsoft Build of OpenJDK](/java/openjdk/download). Ensure that your JAVA_HOME environment variable is set correctly in the shells in which you run the commands.
  - [Maven](https://maven.apache.org/download.cgi) 3.5.0 or higher.
  - Ensure that you have the zip/unzip utility installed; use `zip/unzip -v` to test if `zip/unzip` works.

## Deploy WLS on AKS

The steps in this section direct you to deploy WLS on AKS in the simplest possible way. WLS on AKS offers a broad and deep selection of Azure integrations. For more information, see [What are solutions for running Oracle WebLogic Server on the Azure Kubernetes Service?](/azure/virtual-machines/workloads/oracle/weblogic-aks)

The following steps show you how to find the WLS on AKS offer and fill out the **Basics** pane.

1. In the search bar at the top of the Azure portal, enter *weblogic*. In the auto-suggested search results, in the **Marketplace** section, select **WebLogic Server on AKS**.

   :::image type="content" source="media/howto-deploy-java-wls-app/marketplace-search-results.png" alt-text="Screenshot of the Azure portal showing WLS in search results." lightbox="media/howto-deploy-java-wls-app/marketplace-search-results.png":::

   You can also go directly to the [WebLogic Server on AKS](https://aka.ms/wlsaks) offer.

1. On the offer page, select **Create**.
1. On the **Basics** pane, ensure the value shown in the **Subscription** field is the same one that you logged in Azure. Make sure you have the roles listed in the prerequisites section.

   :::image type="content" source="media/howto-deploy-java-wls-app/portal-start-experience.png" alt-text="Screenshot of the Azure portal showing WebLogic Server on AKS." lightbox="media/howto-deploy-java-wls-app/portal-start-experience.png":::

1. You must deploy the offer in an empty resource group. In the **Resource group** field, select **Create new** and then fill in a value for the resource group. Because resource groups must be unique within a subscription, pick a unique name. An easy way to have unique names is to use a combination of your initials, today's date, and some identifier - for example, `ejb0723wls`.
1. Under **Instance details**, select the region for the deployment. For a list of Azure regions where AKS is available, see [AKS region availability](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service).
1. Under **Credentials for WebLogic**, leave the default value for **Username for WebLogic Administrator**.
1. Fill in `wlsAksCluster2022` for the **Password for WebLogic Administrator**. Use the same value for the confirmation and **Password for WebLogic Model encryption** fields.
1. Scroll to the bottom of the **Basics** pane and notice the helpful links for documentation, community support, and how to report problems.
1. Select **Next**.

The following steps show you how to start the deployment process.

1. Scroll to the section labeled **Provide an Oracle Single Sign-On (SSO) account**. Fill in your Oracle SSO credentials from the preconditions.

   :::image type="content" source="media/howto-deploy-java-wls-app/configure-single-sign-on.png" alt-text="Screenshot of the Azure portal showing the configure sso pane." lightbox="media/howto-deploy-java-wls-app/configure-single-sign-on.png":::

1. Follow the steps in the info box starting with **Before moving forward, you must accept the Oracle Standard Terms and Restrictions.**

1. Depending on whether or not the Oracle SSO account has an Oracle support entitlement, select the appropriate option for **Select the type of WebLogic Server Images.**. If the account has a support entititlement, select **Patched WebLogic Server Images**. Otherwise select **General WebLogic Server Images**.

1. Leave the value in **Select desired combination of WebLogic Server...** at its default value. You have a broad range of choices for WLS, JDK, and OS version.

1. In the **Application** section, next to **Deploy an application?**, select **No**.

The following steps make it so the WLS admin console and the sample app are exposed to the public Internet with a built-in Application Gateway ingress add-on. For a more information, see [What is Application Gateway Ingress Controller?](/azure/application-gateway/ingress-controller-overview).

:::image type="content" source="media/howto-deploy-java-wls-app/configure-load-balancing.png" alt-text="Screenshot of the Azure portal showing the simplest possible load balancer configuration on the Create Oracle WebLogic Server on Azure Kubernetes Service page." lightbox="media/howto-deploy-java-wls-app/configure-load-balancing.png":::

1. Select **Next** to see the **TLS/SSL** pane.
1. Select **Next** to see the **Load balancing** pane.
1. Next to **Load Balancing Options**, select **Application Gateway Ingress Controller**.
1. Under the **Application Gateway Ingress Controller**, you should see all fields pre-populated with the defaults for **Virtual network** and **Subnet**. Leave the default values.
1. For **Create ingress for Administration Console**, select **Yes**.

    :::image type="content" source="media/howto-deploy-java-wls-app/configure-appgateway-ingress-admin-console.png" alt-text="Screenshot of the Azure portal showing Application Gateway Ingress Controllor configuration on the Create Oracle WebLogic Server on Azure Kubernetes Service page." lightbox="media/howto-deploy-java-wls-app/configure-appgateway-ingress-admin-console.png":::

1. Leave default values for other fields.
1. Select **Review + create**. Ensure the validation does not fail. If it fails, fix any validation problems, then select **Review + create** again.
1. Select **Create**.
1. Track the progress of the deployment on the **Deployment is in progress** page.

Depending on network conditions and other activity in your selected region, the deployment may take up to 50 minutes to complete.

You can perform the steps in the section [Create an Azure SQL Database](#create-an-azure-sql-database) while you wait. Return to this section when you have completed creating the database.

## Examine the deployment output

The steps in this section show you how to verify that the deployment has successfully completed.

If you navigated away from the **Deployment is in progress** page, the following steps will show you how to get back to that page. If you're still on the page that shows **Your deployment is complete**, you can skip to the steps after the image below.

1. In the upper left of any portal page, select the hamburger menu and select **Resource groups**.
1. In the box with the text **Filter for any field**, enter the first few characters of the resource group you created previously. If you followed the recommended convention, enter your initials, then select the appropriate resource group.
1. In the left navigation pane, in the **Settings** section, select **Deployments**. You'll see an ordered list of the deployments to this resource group, with the most recent one first.
1. Scroll to the oldest entry in this list. This entry corresponds to the deployment you started in the preceding section. Select the oldest deployment, as shown in the following screenshot.

   :::image type="content" source="media/howto-deploy-java-wls-app/resource-group-deployments.png" alt-text="Screenshot of the Azure portal showing the resource group deployments list." lightbox="media/howto-deploy-java-wls-app/resource-group-deployments.png":::

1. In the left panel, select **Outputs**. This list shows the output values from the deployment. Useful information is included in the outputs.
1. The **adminConsoleExternalUrl** value is the fully qualified, public Internet visible link to the WLS admin console for this AKS cluster. Select the copy icon next to the field value to copy the link to your clipboard. Save this value aside for later.
1. The **clusterExternalUrl**  value is the fully qualified, public Internet visible link to the sample app deployed in WLS on this AKS cluster. Select the copy icon next to the field value to copy the link to your clipboard. Save this value aside for later.
1. The **shellCmdtoOutputWlsImageModelYaml** value is the base64 string of WDT model that built in the container image. Save this value aside for later.
1. The **shellCmdtoOutputWlsImageProperties** value is base64 string of WDT model properties that built in the container image. Save this value aside for later. 
1. The **shellCmdtoConnectAks** value is the Azure CLI command to connect to this specific AKS cluster. This lets you use `kubectl` to administer the cluster.

The other values in the outputs are beyond the scope of this article, but are explained in detail in the [WebLogic on AKS user guide](https://aka.ms/wls-aks-docs).

## Create an Azure SQL Database

[!INCLUDE [create-an-azure-sql-database](includes/jakartaee/create-an-azure-sql-database.md)]

2. Create schema for the sample application. Follow [Query the database](/azure/azure-sql/database/single-database-create-quickstart#query-the-database) to open **Query editor** pane. Enter and run the following query.

   ```sql
   CREATE TABLE COFFEE (ID NUMERIC(19) NOT NULL, NAME VARCHAR(255) NULL, PRICE FLOAT(32) NULL, PRIMARY KEY (ID));
   CREATE TABLE SEQUENCE (SEQ_NAME VARCHAR(50) NOT NULL, SEQ_COUNT NUMERIC(28) NULL, PRIMARY KEY (SEQ_NAME));
   ```

   After a successful run, you should see the message **Query succeeded: Affected rows: 0**. If you do not see this message, troubleshoot and resolve the problem before proceeding.

The database, tables, AKS cluster and WLS cluster have been created. If you want, you can explore the admin console by opening a browser and navigating to the address of **adminConsoleExternalUrl**. Sign in with the values you entered during the WLS on AKS deployment.

You can proceed to preparing AKS to host your WebLogic application.

## Configure and deploy the sample application

The offer provisions WLS cluster via [model in image](https://oracle.github.io/weblogic-kubernetes-operator/samples/domains/model-in-image/). While, currently, the WLS cluster has no application deployed.

This section updates the WLS cluster by deploying a sample application using [auxiliary image](https://oracle.github.io/weblogic-kubernetes-operator/managing-domains/model-in-image/auxiliary-images/#using-docker-to-create-an-auxiliary-image).

### Check out the application

Clone the sample code for this guide. The sample is on [GitHub](https://github.com/microsoft/weblogic-on-azure).

You'll use `javaee/weblogic-cafe/`. Here's the file structure of the application. 
```text
weblogic-cafe
├── pom.xml
└── src
    └── main
        ├── java
        │   └── cafe
        │       ├── model
        │       │   ├── CafeRepository.java
        │       │   └── entity
        │       │       └── Coffee.java
        │       └── web
        │           ├── rest
        │           │   └── CafeResource.java
        │           └── view
        │               └── Cafe.java
        ├── resources
        │   ├── META-INF
        │   │   └── persistence.xml
        │   └── cafe
        │       └── web
        │           ├── messages.properties
        │           └── messages_es.properties
        └── webapp
            ├── WEB-INF
            │   ├── beans.xml
            │   ├── faces-config.xml
            │   └── web.xml
            ├── index.xhtml
            └── resources
                └── components
                    └── inputPrice.xhtml
```

Clone the repository.

```bash
cd <parent-directory-to-check-out-sample-code>
export BASE_DIR=$PWD
git clone --single-branch https://github.com/microsoft/weblogic-on-azure.git --branch 20240201 $BASE_DIR/weblogic-on-azure
```

If you see a message about being in "detached HEAD" state, this message is safe to ignore. It just means you have checked out a tag.

Build `javaee/weblogic-cafe/`.

```bash
mvn clean package --file $BASE_DIR/weblogic-on-azure/javaee/weblogic-cafe/pom.xml
```

The package should be successfully generated and located at `$BASE_DIR/weblogic-on-azure/javaee/weblogic-cafe/target/weblogic-cafe.war`. If you don't see the package, you must troubleshoot and resolve the issue before you continue.

### Use Docker to create an auxiliary image

The steps in this section show you how to build an auxiliary image, including Model in Image model files, your application, the JDBC driver archive file, and the WebLogic Deploy Tooling installation.

An auxiliary image is a Docker container image containing your app and configuration. The WebLogic Kubernetes Operator combines your auxiliary image with the `domain.spec.image` in the AKS cluster which contains the WebLogic Server, JDK, and operating system. To learn more about auxiliary images, see [Auxiliary images](https://oracle.github.io/weblogic-kubernetes-operator/managing-domains/model-in-image/auxiliary-images/) in the Oracle documentation.

This section requires a Linux terminal with Azure CLI and kubectl installed.

1. Create a directory to stage the models and application.

   ```bash
   mkdir -p ${BASE_DIR}/mystaging/models
   cd ${BASE_DIR}/mystaging/models
   ```

   Paste the value of **shellCmdtoOutputWlsImageModelYaml** you saved from the deployment outputs. You get a file `${BASE_DIR}/mystaging/models/model.yaml`. The command will look similar to the following.
    
   ```bash
   echo -e IyBDb3B5cmlna...Cgo= | base64 -d > model.yaml
   ```
    
   The content of *model.yaml* is similar to the following text.

   ```yaml
   # Copyright (c) 2020, 2021, Oracle and/or its affiliates.
   # Licensed under the Universal Permissive License v 1.0 as shown at https://oss.oracle.com/licenses/upl.

   # Based on ./kubernetes/samples/scripts/create-weblogic-domain/model-in-image/model-images/model-in-image__WLS-v1/model.10.yaml
   # in https://github.com/oracle/weblogic-kubernetes-operator.

   domainInfo:
     AdminUserName: "@@SECRET:__weblogic-credentials__:username@@"
     AdminPassword: "@@SECRET:__weblogic-credentials__:password@@"
     ServerStartMode: "prod"

   topology:
     Name: "@@ENV:CUSTOM_DOMAIN_NAME@@"
     ProductionModeEnabled: true
     AdminServerName: "admin-server"
     Cluster:
       "cluster-1":
         DynamicServers:
           ServerTemplate: "cluster-1-template"
           ServerNamePrefix: "@@ENV:MANAGED_SERVER_PREFIX@@"
           DynamicClusterSize: "@@PROP:CLUSTER_SIZE@@"
           MaxDynamicClusterSize: "@@PROP:CLUSTER_SIZE@@"
           MinDynamicClusterSize: "0"
           CalculatedListenPorts: false
     Server:
       "admin-server":
         ListenPort: 7001
     ServerTemplate:
       "cluster-1-template":
         Cluster: "cluster-1"
         ListenPort: 8001
     SecurityConfiguration:
       NodeManagerUsername: "@@SECRET:__weblogic-credentials__:username@@"
       NodeManagerPasswordEncrypted: "@@SECRET:__weblogic-credentials__:password@@"

   resources:
     SelfTuning:
       MinThreadsConstraint:
         SampleMinThreads:
           Target: "cluster-1"
           Count: 1
       MaxThreadsConstraint:
         SampleMaxThreads:
           Target: "cluster-1"
           Count: 10
       WorkManager:
         SampleWM:
           Target: "cluster-1"
           MinThreadsConstraint: "SampleMinThreads"
           MaxThreadsConstraint: "SampleMaxThreads"
   ```

1. In a similar way, paste the value of **shellCmdtoOutputWlsImageProperties**. You get a file `${BASE_DIR}/mystaging/models/model.properties`. The command will look similar to the following.

   ```bash
   echo -e IyBDb3B5cml...pFPTUK | base64 -d > model.properties
   ```
    
   The content of *model.properties* is similar to the following text.

   ```text
   # Copyright (c) 2021, Oracle Corporation and/or its affiliates.
   # Licensed under the Universal Permissive License v 1.0 as shown at https://oss.oracle.com/licenses/upl.

   # Based on ./kubernetes/samples/scripts/create-weblogic-domain/model-in-image/model-images/model-in-image__WLS-v1/model.10.properties
   # in https://github.com/oracle/weblogic-kubernetes-operator.

   CLUSTER_SIZE=5
   ```

1. Create the application model file.

   Copy `weblogic-cafe.war` and save it to `wlsdeploy/applications`.

   ```bash
   mkdir -p ${BASE_DIR}/mystaging/models/wlsdeploy/applications
   cp $BASE_DIR/weblogic-on-azure/javaee/weblogic-cafe/target/weblogic-cafe.war ${BASE_DIR}/mystaging/models/wlsdeploy/applications/weblogic-cafe.war
   ```

   Create the application model file with the following content. Save the model file to `${BASE_DIR}/mystaging/models/appmodel.yaml`.

   ```bash
   cat <<EOF >appmodel.yaml
   appDeployments:
     Application:
       weblogic-cafe:
         SourcePath: 'wlsdeploy/applications/weblogic-cafe.war'
         ModuleType: ear
         Target: 'cluster-1'
   EOF
   ```

1. Download and install Microsoft SQL Server JDBC driver to `wlsdeploy/externalJDBCLibraries`.

   ```bash
   export DRIVER_VERSION="10.2.1.jre8"
   export MSSQL_DRIVER_URL="https://repo.maven.apache.org/maven2/com/microsoft/sqlserver/mssql-jdbc/${DRIVER_VERSION}/mssql-jdbc-${DRIVER_VERSION}.jar"

   mkdir ${BASE_DIR}/mystaging/models/wlsdeploy/externalJDBCLibraries
   curl -m 120 -fL ${MSSQL_DRIVER_URL} -o ${BASE_DIR}/mystaging/models/wlsdeploy/externalJDBCLibraries/mssql-jdbc-${DRIVER_VERSION}.jar
   ```

   Next, create the database connection model with the following content. Save the model file to `${BASE_DIR}/mystaging/models/dbmodel.yaml`. The model uses placeholder (secret `sqlserver-secret`) for database username, password, and Url. Make sure the following fields are set correcly. The following model names the resource with **jdbc/WebLogicCafeDB**.

   | Item Name | Field | Value |
   |--------------------|----------------------------|----------------|
   | JNDI name | `resources.JDBCSystemResource.<resource-name>.JdbcResource.JDBCDataSourceParams.JNDIName`  | `jdbc/WebLogicCafeDB` |
   | Driver name | `resources.JDBCSystemResource.<resource-name>.JDBCDriverParams.DriverName` | `com.microsoft.sqlserver.jdbc.SQLServerDriver` |
   | Database Url | `resources.JDBCSystemResource.<resource-name>.JDBCDriverParams.URL`  | `@@SECRET:sqlserver-secret:url@@` |
   | Database password | `resources.JDBCSystemResource.<resource-name>.JDBCDriverParams.PasswordEncrypted` | `@@SECRET:sqlserver-secret:password@@` |
   | Database username | `resources.JDBCSystemResource.<resource-name>.JDBCDriverParams.Properties.user.Value`  |  `'@@SECRET:sqlserver-secret:user@@'` |

   ```bash
   cat <<EOF >dbmodel.yaml
   resources:
     JDBCSystemResource:
       jdbc/WebLogicCafeDB:
         Target: 'cluster-1'
         JdbcResource:
           JDBCDataSourceParams:
             JNDIName: [
               jdbc/WebLogicCafeDB
             ]
             GlobalTransactionsProtocol: None
           JDBCDriverParams:
             DriverName: com.microsoft.sqlserver.jdbc.SQLServerDriver
             URL: '@@SECRET:sqlserver-secret:url@@'
             PasswordEncrypted: '@@SECRET:sqlserver-secret:password@@'
             Properties:
               user:
                 Value: '@@SECRET:sqlserver-secret:user@@'
           JDBCConnectionPoolParams:
             TestTableName: SQL SELECT 1
             TestConnectionsOnReserve: true
   EOF
   ```

1. Create application archive file using `zip` command. Remove `wlsdeploy` folder as you'll not use it anymore.

   ```bash
   cd ${BASE_DIR}/mystaging/models
   zip -r archive.zip wlsdeploy

   rm -f -r wlsdeploy
   ```

1. Download and install [WebLogic Deploy Tooling](https://oracle.github.io/weblogic-deploy-tooling/)(WDT) in the staging directory and remove its weblogic-deploy/bin/*.cmd files, which are not used in UNIX environments:

   ```bash
   cd ${BASE_DIR}/mystaging
   curl -m 120 -fL https://github.com/oracle/weblogic-deploy-tooling/releases/latest/download/weblogic-deploy.zip -o weblogic-deploy.zip

   unzip weblogic-deploy.zip -d .
   rm ./weblogic-deploy/bin/*.cmd
   ```

   Remove the WDT installer.

   ```bash
   rm weblogic-deploy.zip
   ```

1. Build an auxiliary image using docker.

   ```bash
   cd ${BASE_DIR}/mystaging
   cat <<EOF >Dockerfile
   FROM busybox
   ARG AUXILIARY_IMAGE_PATH=/auxiliary
   ARG USER=oracle
   ARG USERID=1000
   ARG GROUP=root
   ENV AUXILIARY_IMAGE_PATH=\${AUXILIARY_IMAGE_PATH}
   RUN adduser -D -u \${USERID} -G \$GROUP \$USER
   # ARG expansion in COPY command's --chown is available in docker version 19.03.1+.
   # For older docker versions, change the Dockerfile to use separate COPY and 'RUN chown' commands.
   COPY --chown=\$USER:\$GROUP ./ \${AUXILIARY_IMAGE_PATH}/
   USER \$USER
   EOF
   ```

   Run the `docker buildx build` command using `${BASE_DIR}/mystaging/Dockerfile`.

   ```bash
   cd ${BASE_DIR}/mystaging
   docker buildx build --platform linux/amd64 --build-arg AUXILIARY_IMAGE_PATH=/auxiliary --tag model-in-image:WLS-v1 .
   ```

   You'll find output as following when you build the image successfully.

   ```text
   [+] Building 12.0s (8/8) FINISHED                                   docker:default
   => [internal] load build definition from Dockerfile                          0.8s
   => => transferring dockerfile: 473B                                          0.0s
   => [internal] load .dockerignore                                             1.1s
   => => transferring context: 2B                                               0.0s
   => [internal] load metadata for docker.io/library/busybox:latest             5.0s
   => [1/3] FROM docker.io/library/busybox@sha256:6d9ac9237a84afe1516540f40a0f  0.0s
   => [internal] load build context                                             0.3s
   => => transferring context: 21.89kB                                          0.0s
   => CACHED [2/3] RUN adduser -D -u 1000 -G root oracle                        0.0s
   => [3/3] COPY --chown=oracle:root ./ /auxiliary/                             1.5s
   => exporting to image                                                        1.3s
   => => exporting layers                                                       1.0s
   => => writing image sha256:2477d502a19dcc0e841630ea567f50d7084782499fe3032a  0.1s
   => => naming to docker.io/library/model-in-image:WLS-v1                      0.2s
   ```

   If you have successfully created the image, then it should now be in your local machine’s Docker repository. For example:

   ```text
   $ docker images model-in-image:WLS-v1
   REPOSITORY       TAG       IMAGE ID       CREATED       SIZE
   model-in-image   WLS-v1    76abc1afdcc6   2 hours ago   8.61MB
   ```

   After the image is created, it should have the WDT executables in /auxiliary/weblogic-deploy, and WDT model, property, and archive files in /auxiliary/models. You can run ls in the Docker image to verify this:

   ```text
   $ docker run -it --rm model-in-image:WLS-v1 find /auxiliary -maxdepth 2 -type f -print
   /auxiliary/models/dbmodel.yaml
   /auxiliary/models/archive.zip
   /auxiliary/models/model.properties
   /auxiliary/models/model.yaml
   /auxiliary/weblogic-deploy/VERSION.txt
   /auxiliary/weblogic-deploy/LICENSE.txt
   /auxiliary/Dockerfile
   ```

1. Push the auxiliary image to Azure Container Registry. 

   - Open Azure portal and go to the resource group that was provisioned in [Deploy WSL on AKS](#deploy-wls-on-aks).
   - Select the resource of type **Container registry** from resource list.
   - Hover the mouse over the value next to **Login server** and select the copy icon to the right of the text. Set this as the value of the `ACR_LOGIN_SERVER` environment variable.
   
      ```bash
      export ACR_LOGIN_SERVER=<value-from-clipboard>
      ```

   - Run the following commands to tag and push the image. Make sure Docker is running before executing these commands.

      ```bash
      export ACR_NAME=$(echo ${ACR_LOGIN_SERVER} | cut -d '.' -f 1)
      az acr login -n $ACR_NAME
      docker tag model-in-image:WLS-v1 $ACR_LOGIN_SERVER/wlsaks-auxiliary-image:1.0
      docker push $ACR_LOGIN_SERVER/wlsaks-auxiliary-image:1.0
      ```

      You can run `az acr repository show` to test if the image is push to remote repository successfully.

      ```bash
      az acr repository show --name ${ACR_NAME} --image wlsaks-auxiliary-image:1.0
      ```

      You'll find the output is similar to the following content.

      ```text
      {
        "changeableAttributes": {
          "deleteEnabled": true,
          "listEnabled": true,
          "readEnabled": true,
          "writeEnabled": true
        },
        "createdTime": "2024-01-24T06:14:19.4546321Z",
        "digest": "sha256:a1befbefd0181a06c6fe00848e76f1743c1fecba2b42a975e9504ba2aaae51ea",
        "lastUpdateTime": "2024-01-24T06:14:19.4546321Z",
        "name": "1.0",
        "quarantineState": "Passed",
        "signed": false
      }
      ```

### Apply the auxiliary image

In the previous steps, you created the auxiliary image including models and WDT. Before you apply the auxiliary image to the WLS cluster, create the secret for datasource Url, username and password. The secret is used as part of the placeholder in the *dbmodel.yaml*.

1. Connect to the AKS cluster.

   - In your shell, paste the value of the **shellCmdtoConnectAks** you saved aside previously. Execute the command. This will look similar to the following.
   
      ```bash
      az account set --subscription <redacted>; az aks get-credentials --resource-group ejb010201wls --name wlsonaks2mkgvjqpy4cl4
      ```
      
      You should see output similar to the following. If you don't see this output, troubleshoot and resolve the problem before continuing.
      
      ```bash
      Merged "wlsonaks2mkgvjqpy4cl4" as current context in /Users/<username>/.kube/config
      ```
   
1. Create secret for datasource connection.

    This article uses secret name `sqlserver-secret` for the secret of the datasource connection. Run the following command to create the [Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/). If you use a different name, make sure the value is the same with that used in *dbmodel.yaml*. Make sure variable **DB_CONNECTION_STRING**, **DB_USER** and **DB_PASSWORD**  for database connection are set correctly.

   | Variable | Value | Example |
   |----------|-------|---------|
   |`DB_CONNECTION_STRING` |  The connection string of SQL server. You can obtain it from portal. | `jdbc:sqlserver://sqlserverforwlsaks.database.windows.net:1433;database=wlsaksquickstart0125`
   | `DB_USER` | The username to login the SQL server. | `welogic@sqlserverforwlsaks` |
   | `DB_PASSWORD` | The password to login the sQL server. | `Secret123456` |


   Follow these steps to get the values for the variables.
   
   1. Visit the SQL database resource in the portal.
   
   1. In the left navigation panel, under **Settings**, select **Connection strings**.
   
   1. Select the **JDBC** tab.
   
   1. Select the copy icon.
   
   1. The `DB_CONNECTION_STRING` value is the portion of the string from `jdbc:` up to but not including `;user=azureuser`.
   
   1. The `DB_USER` value is the portion of the string from `azureuser` up to but not including `;password={your_password_here}`.
   
   1. The `DB_PASSWORD` value is the value you entered when you created the database.

   ```bash
   export DB_CONNECTION_STRING=<example-jdbc:sqlserver://sqlserverforwlsaks.database.windows.net:1433;database=wlsaksquickstart0125>
   export DB_USER=<example-welogic@sqlserverforwlsaks>
   export DB_PASSWORD=<example-Secret123456>
   export WLS_DOMAIN_NS=sample-domain1-ns
   export WLS_DOMAIN_UID=sample-domain1
   export SECRET_NAME=sqlserver-secret

   kubectl -n ${WLS_DOMAIN_NS} create secret generic \
     ${SECRET_NAME} \
     --from-literal=password="${DB_PASSWORD}" \
     --from-literal=url="${DB_CONNECTION_STRING}" \
     --from-literal=user="${DB_USER}"

   kubectl -n ${WLS_DOMAIN_NS} label secret \
     ${SECRET_NAME} \
     weblogic.domainUID=${WLS_DOMAIN_UID}
   ```
   
   You must see the following output before continuing. If you do not see this output, troubleshoot and resolve the problem before continuing.
   
   ```bash
   secret/sqlserver-secret created
   secret/sqlserver-secret labeled
   ```

2. Apply the auxiliary image by patching the domain custom resource definition (CRD).

   The auxiliary image is defined in `spec.configuration.model.auxiliaryImages`, as the following snippet shows. For more information, see [auxiliary images](https://oracle.github.io/weblogic-kubernetes-operator/managing-domains/model-in-image/auxiliary-images/).

   ```yaml
   spec:
     clusters:
     - name: sample-domain1-cluster-1
     configuration:
       model:
         auxiliaryImages:
         - image: wlsaksacrafvzeyyswhxek.azurecr.io/wlsaks-auxiliary-image:1.0
           imagePullPolicy: IfNotPresent
           sourceModelHome: /auxiliary/models
           sourceWDTInstallHome: /auxiliary/weblogic-deploy
   ```

   Run `kubectl patch` command increase the `restartVersion` and apply the auxiliary image with the following definition in domain CRD.

   ```bash
   export VERSION=$(kubectl -n ${WLS_DOMAIN_NS} get domain ${WLS_DOMAIN_UID} -o=jsonpath='{.spec.restartVersion}' | tr -d "\"")
   ```

   ```bash
   export VERSION=$((VERSION+1))

   cat <<EOF >patch-file.json
   [
     {
       "op": "replace",
       "path": "/spec/restartVersion",
       "value": "${VERSION}"
     },
     {
       "op": "add",
       "path": "/spec/configuration/model/auxiliaryImages",
       "value": [{"image": "$ACR_LOGIN_SERVER/wlsaks-auxiliary-image:1.0", "imagePullPolicy": "IfNotPresent", "sourceModelHome": "/auxiliary/models", "sourceWDTInstallHome": "/auxiliary/weblogic-deploy"}]
     },
     {
       "op": "add",
       "path": "/spec/configuration/secrets",
       "value": ["${SECRET_NAME}"]
     }
   ]
   EOF

   kubectl -n ${WLS_DOMAIN_NS} patch domain ${WLS_DOMAIN_UID} \
     --type=json \
     --patch-file patch-file.json

   kubectl get pod -n ${WLS_DOMAIN_NS} -w
   ```
   
   Wait until the admin server and managed servers show the following values before proceeding. It may take a five - ten minutes for the system to reach this state. Here is a high level overview of what is happening while you wait.
   
   - You'll see the `sample-domain1-introspector-zbtbz` running first. This software looks for changes to the domain custom resource so it can take the necessary actions on the Kubernetes cluster.
   - When changes are detected, the domain introspector kills and starts new pods to roll out the changes.
   - Next, you'll see the `sample-domain1-admin-server` pod terminate and restart.
   - Then, you'll see the two managed servers terminate and restart.
   - Only when all three pods show the `1/1 Running` state, is it ok to proceed.
   
   ```bash
   NAME                             READY   STATUS    RESTARTS   AGE
   sample-domain1-admin-server      1/1     Running   0          20m
   sample-domain1-managed-server1   1/1     Running   0          19m
   sample-domain1-managed-server2   1/1     Running   0          18m
   ```

## Verify the functionality of the deployment

The following steps show you how to verify the functionality of the deployment by viewing the WLS admin console and the sample app.

1. Paste the value for **adminConsoleExternalUrl** in an Internet-connected web browser. You should see the familiar WLS admin console login screen.

1. Sign in with the username `weblogic` and the password you entered when deploying WLS from the portal. Recall this was `wlsAksCluster2022`.

1. In the **Domain Structure** box, select **Deployments**.

1. In the **Deployments** table, there should be one row. The name should be the same value as the `Application` in your `appmodel.yaml` file. Select the name.

1. In the **Settings** panel, select the **Testing** tab.

1. Select **weblogic-cafe**.

1. In the **Settings for weblogic-cafe** panel, select the **Testing** tab.

1. Expand the `+` icon next to **weblogic-cafe**.  You should see values similar to these.

   :::image type="content" source="media/howto-deploy-java-wls-app/weblogic-cafe-deployment.png" alt-text="Screenshot of WLS admin login screen." border="false":::

   > [!NOTE]
   > The hyperlinks in the **Test Point** column are not selectable because we did notconfigure the admin console with the external URL on which it is running. This article shows the WLS admin console merely by way of demonstration. Don't use the WLS admin console for any durable configuration changes when running WLS on AKS. The cloud-native design of WLS on AKS requires that any durable configuration must be represented in the initial docker images or applied to the running AKS cluster using CI/CD techniques such as updating the model, as described in the [Oracle documentation](https://aka.ms/wls-aks-docs-update-model).

1. Understand the `context-path` of the sample app you deployed. If you deployed the recommended sample app, the `context-path` is `weblogic-cafe`.
1. Construct a fully qualified URL for the sample app by appending the `context-path` to the value of **clusterExternalUrl**. If you deployed the recommended sample app, the fully qualified URL will be something like `http://wlsgw202401-wls-aks-domain1.eastus.cloudapp.azure.com/weblogic-cafe/`.
1. Paste the fully qualified URL in an Internet-connected web browser. If you deployed the recommended sample app, you should see results similar to the following screenshot.

   :::image type="content" source="media/howto-deploy-java-wls-app/weblogic-cafe-app.png" alt-text="Screenshot of test web app." border="false":::

## Clean up resources

To avoid Azure charges, you should clean up unnecessary resources. When you no longer need the cluster, use the [az group delete](/cli/azure/group#az-group-delete) command. The following command will remove the resource group, container service, container registry, and all related resources.

```azurecli
az group delete --name <resource-group-name> --yes --no-wait
az group delete --name <db-resource-group-name> --yes --no-wait
```

## Next steps

Learn more about running WLS on AKS or virtual machines by following these links:

> [!div class="nextstepaction"]
> [WLS on AKS](/azure/virtual-machines/workloads/oracle/weblogic-aks)

> [!div class="nextstepaction"]
> [Migrate WebLogic Server applications to Azure Kubernetes Service](/azure/developer/java/migration/migrate-weblogic-to-azure-kubernetes-service)

> [!div class="nextstepaction"]
> [WLS on virtual machines](/azure/virtual-machines/workloads/oracle/oracle-weblogic)
