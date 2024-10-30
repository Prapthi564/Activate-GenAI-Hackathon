# Challenge 03: Deploy NVIDIA NIM to Azure

### Estimated Time: 90 minutes

## Introduction

This lab guides participants through deploying NVIDIA NeMo Inference Manager (NIM) on Azure, which facilitates rapid development and deployment of sophisticated AI models for various applications. NVIDIA NIM is a model deployment and management solution that enables efficient utilization of NVIDIA GPUs for AI model inference. This deployment involves using Azure's powerful cloud infrastructure to host NIM containers, which simplifies the integration of AI models into production environments by managing model lifecycle, deployment, and scaling in Azure Machine Learning (AML) services.

Participants will begin by creating an NVIDIA account to generate an API key, essential for accessing NVIDIA’s NGC resources, such as container images for NeMo models. Then, the lab covers setting up Azure Container Registry (ACR) for storing these images, configuring Git Bash to interact with Azure, and updating relevant configuration files within Visual Studio Code. After this setup, participants will deploy the NIM container through Azure Machine Learning, which will handle the GPU-based model deployment and expose it through a managed online endpoint.

## Solution Guide

### Task 1: Generate NGC API KEY

> **Note:** This process of getting NGC API key is no longer recommended. Participant is recommended to get API key through build.nvidia.com as mentioned in the Scenario page.

The NVIDIA API key is a unique identifier used to authenticate requests to NVIDIA's APIs, such as the NGC (NVIDIA GPU Cloud) services. This key allows developers to access various resources, including pre-trained models, GPU-accelerated software, and container images. Obtaining an API key typically involves creating an account on NVIDIA's developer portal and generating the key within the account settings. It is important to keep this key secure, as it grants access to your NVIDIA resources and can be used for billing purposes.

1. Please log in to [nvidia](https://ngc.nvidia.com/signin) account using your credentials.

   >**Note:** If you don't have an account, simply enter your work email account and submit, and you'll be redirected to the account creation page to provide the necessary details and set up your account.

    ![](../../Coach/media/nvidia1a.png)

1. On the **Create an Nvidia Cloud Account** page enter the **Account Name** as per your preference.

   ![](../../Coach/media/nvidia1c.png)

1. On the **Set Your Profile** page provide the details such as **First Name**, **Last Name**, **Location**, **Job Role**, **Organization**, **Industry Segment(s)**, **Area of Interest** and then click on **Submit**.

   ![](../../Coach/media/nvidia1b.png)

1. Once your account is created or you've successfully logged in, navigate to the Developer Portal.

1. Navigate to [NVIDIA Devloper Portal](https://developer.nvidia.com/developer-program?ncid=pa-srch-goog-746366&_bt=699127533347&_bk=join%20nvidia%20developer%20program&_bm=b&_bn=g&_bg=161777607269&gad_source=1&gclid=Cj0KCQjwxsm3BhDrARIsAMtVz6MYF3MjayqdjwpkljkJ5ZSVv8IqKIExaNHyiPgUykfiJPplssYqyeIaAlNDEALw_wcB) and Click on the **join** option from the top.

   ![](../../Coach/media/nvidia1.png)

1. Log in to your account using your credentials to access the platform and its features. If you encounter any issues, ensure your login details are correct or reset your password if needed.

   ![](../../Coach/media/nvidia2.png)

1. If prompted, fill out the form to complete your profile setup.

   ![](../../Coach/media/nvidia3.png)

1. Navigate back to the your **Nvidia Account** Click on **Account** at the top of the page and navigate to the **Setup** section.

   ![](../../Coach/media/nvidia4.png)

1. Click on **Generate API Key** to create a new key for accessing the necessary services.

   ![](../../Coach/media/nvidia5.png)

1. From the top, click on **+ Generate API Key** to create a new API key.

   ![](../../Coach/media/nvidia8.png)

1. Click on **Confirm** to generate your new API key.

   ![](../../Coach/media/nvidia9.png)

1. Carefully copy your generated API key, essential for accessing various services and features paste the API key in the notebook. Ensure you store it securely, as it may not be displayed again after you leave the page.

   ![](../../Coach/media/nvidia7.png)

## Accessing the Azure portal

>**Important**: You can find the Username and Password within the environment by navigating to the **Environment** **(1)** tab in the left pane then copy the **Azure Username** **(2)** and **Azure Password** **(3)**, which will be required for signing into the Azure portal in later steps and you can record the **Deployment Id** **(4)**, which can be used to provide a unique name to the resources during deployment.

   >**Note**: Numbers and ID's values may vary kindly ignore values in screenshots and copy values from **Environment** tab.

 ![](../media/Active-image19.png)
 ![](../media/Active-image(20).png)

1. To access the Azure portal, within labvm open **Microsoft Edge** and browser to the [Azure Portal](https://portal.azure.com/).

1. On the **Sign into Microsoft Azure tab**, you will see a login screen. Enter the following email/username, and then click on **Next**.
   
   - **Email/Username:** <inject key="AzureAdUserEmail"></inject>

     ![](../media/Active-image1.png)

1. Now enter the following password and click on **Sign in**.

   - **Password:** <inject key="AzureAdUserPassword"></inject>

      ![](../media/Active-image2.png)

1. When **Action Required** window pop up click on **Ask Later**.

    ![](../media/Active-image3.png)
   
1. If you see the pop-up **Stay Signed in?**, click **No**.

    ![](../media/Active-image4.png)

1. If a **Welcome to Microsoft Azure** pop-up window appears, click **Cancel** to skip the tour.

    ![](../media/Active-image5.png)

### Task 2: Create Container Registry

Azure Container Registry (ACR) is a managed Docker container registry service that allows you to store and manage private Docker container images and artifacts in Azure. It provides a secure and scalable solution for building, deploying, and managing containerized applications, enabling seamless integration with Azure services.

1. In the search bar of the Azure portal, type **Container registries** (1). From the search results, select **Container registries** (2) to access the container registry management section.

   ![](../../Coach/media/cr1.png)

1. Click on **+ Create**.

1. On the **Basics** tab of Create **Container Registry**, provide details as mentioned in the table below and select **Review + create** (5) at the bottom of the page and subsequently click on **Create**.

    | Setting | Action |
    | -- | -- |
    | **Subscription** | Default |
    | **Resource Group** | **Activate-GenAI** (1) |
    | **Registry name** | **amlregistry<inject key="DeploymentID" enableCopy="false"/>** (2) |
    | **Location** | **East US 2** (Choose the same location where the resource group) (3) |
    | **Pricing plan** | **Standard** (4) |

   ![](../../Coach/media/aml1.png)

   ![](../../Coach/media/aml2.png)

1. Once the deployment is completed, click on **Go to resource**.

1. From the Overview page copy the subscription ID and paste the  subscription ID into the notebook you will use later use.

   ![](../../Coach/media/aml3.png)

### Task 3: Setup Git Bash Environment

1. In the **LabVM**, click on the Start menu and search for **Git Bash** **(1)**. Once you find it, right-click on **Git Bash** **(2)** and select **Run as Administrator** **(3)** to launch Git Bash with elevated privileges.

   ![](../media/git-bash-run.png)

1. Run the following command, This command downloads the latest version of jq, a lightweight and flexible command-line JSON processor, and saves it as an executable file named `jq.exe` in the `/usr/bin/` directory, making it accessible for command-line use.

   ```
   curl -L -o /usr/bin/jq.exe https://github.com/stedolan/jq/releases/latest/download/jq-win64.exe
   ```

1. Now, install the ml extension

   ```
   az extension add -n ml --allow-preview false
   ```

   ```
   az extension update -n ml
   ```

1. Run the help command to verify your installation and see available subcommands:

   ```
   az ml -h
   ```

1. Clone the GitHub repository to your Desktop 

   ```
   cd Desktop
   ```

   ```
   git clone https://github.com/CloudLabsAI-Azure/nim-deploy.git
   ```

1. Use the command below to navigate to the directory:

   ```
   cd nim-deploy/cloud-service-providers/azure/azureml/cli
   ```

Detailed instructions can be found [here](https://github.com/NVIDIA/nim-deploy/tree/main/cloud-service-providers/azure/azureml/cli).

### Task 4: Visual Studio config.sh file update

1. Start **Visual Studio Code** by launching it from your desktop.

   ![](../../Coach/media/vscode1.png)

1. Go to the **Explorer** panel in the upper left corner, click on **Open Folder**, and then select the **nim-deploy** folder from your Desktop.

1. Navigate to the **cli folder**, and then click on **Select Folder** to open it in Visual Studio Code.

   ```
   Desktop\nim-deploy\cloud-service-providers\azure\azureml\cli
   ```

   ![](../../Coach/media/vscode2.png)

1. Select **Yes, I trust the authors**.

   ![](../../Coach/media/vscode3.png)

1. Open the `config.sh` file and update the values as needed. You can refer to the `example_config.sh` file for guidance on the appropriate configurations and settings.

    | Setting | Action |
    | -- | -- |
    | **subscription_id** | **<inject key="SubscriptionID" enableCopy="false"/>** |
    | **resource_group** | **Activate-GenAI**  |
    | **workspace** | **ml-workspace** (Provide the name of workspace you want to create) |
    | **location** | **EastUS2**, **CentralUS** (Choose the same location where the resource group and make sure there is no space between the loaction name) |
    | **ngc_api_key** | Provide the NGC key  |
    | **email_address** | Enter the email from Environmental Details tab  |
    | **acr_registry_name** | **amlregistry<inject key="DeploymentID" enableCopy="false"/>** |
    | **image_name** | **nim-meta-llama-3.1-8b-instruct:latest**|

   ![](../../Coach/media/vscode4.png)

   ![](../../Coach/media/up2.png)

1. Press **Ctrl + S** to save the changes you made to the file.

## Task 5: Create AzureML Deployment of the NIM Container

### Login to Azure with Your Credentials

1. Switch back to the Git Bash terminal.

1. Update your login credentials (**Username** and **Password**) and set the **subscription_id** for your subscription.
   
   - **Subscription Id:** - <inject key="SubscriptionID"></inject>
   - **Username:** <inject key="AzureAdUserEmail"></inject>
   - **Password:** <inject key="AzureAdUserPassword"></inject>

   ```
   source config.sh
   ```
   ```
   az login --user <Username> --password <Password>
   az account set -s ${subscription_id}
   ```
   > **Note:** If you encounter any issues during login, you can execute the following command.

   ```
   az account clear
   az config set core.enable_broker_on_windows=false
   az login --user <Username> --password <Password>
   az account set -s ${subscription_id}
   ```

1. This will prompt an Azure login window; please select your credentials to log in.

###  Task 6: Setup AzureML Workspace

1. Execute the following command to create a new AzureML workspace with the "Azure ML Secrets Reader" role assignment.

   ```cmd
    ./1_set_credentials.sh --create_new_workspace
   ```
   > **Note:** The above command creates a new workspace with the workspace name provided in the config.sh file.

1. You can find the newly created worksapce in azure

   ![](../../Coach/media/u1.png)

###  Task 7: Store NGC API Key for Use in the AzureML Deployment

1. To Store NGC API Key for Use in the AzureML Deployment.You have two options for storing the NGC API Key:

   >**Note :** The NGC API Key needs to be stored within Azure so the AzureML workspace can access it during deployment. The API key is required to pull the correct model from the NGC model  catalog. The key can be provided as a workspace connection to the AzureML workspace.
   
   **Store the Key as a Workspace Connection:**
   Store the NGC API Key as a custom credential using a workspace connection.

   This script stores the NGC API Key as a workspace connection credential and verifies if the provided workspace can access it.

   Run the following script to configure this and verify the connection

   ```cmd
    ./2_provide_ngc_connection.sh
   ```

###  Task 8: Save NIM Container in Your Container Registry

Pull the NIM Docker container for the model specified in the `config.sh` file. Create another Docker container wrapped around the NIM container for deployment in AzureML and push this new container to an Azure container registry that can be accessed by your AzureML endpoint. All required commands are provided in the `3_save_nim_container.sh` script.

1. Run the following command to **save the NIM container in your container registry**.

   ```cmd
   ./3_save_nim_container.sh
   ```
   >**Note:** This action will approximately take around 20-25 Minutes.

1. Navigate to your container registry (**amlregistry**) , Under the service click on the Respositiories select your **nim-meta-llama-3.1-8b-instruct** regiestry, here you will find your image is pushed with the tag name **latest**.

   ![](../../Coach/media/bash2.png)

###  Task 9: Create Managed Online Endpoint

1. Run the following command to **create a managed online endpoint**.

   ```cmd
   ./4_create_endpoint.sh
   ```

   >**Note :** This command creates an endpoint with the name provided in the config.sh file.

###  Task 10: Role Assignment

1. Go to **amlregistry** container regiestry first. navigate to **Access control (IAM)** (1). Click on **+ Add**(2) and choose **Add role assignment** (3). This allows you to assign specific roles to users, groups, or applications, controlling their permissions to manage resources associated with the app service.

   ![](../../Coach/media/bash3.png)

1. In the **Add role assignment** page, under the Role tab, choose **Job function roles** (1). Search and select **AcrPull** (2) within this category, and then click **Next** (3) to proceed.

   ![](../../Coach/media/bash4.png)

1. Next, under the **Members** tab, select **Managed identity** (1) for Assign access to, and then click on **+ Select members** (2). Further, under the **Select managed identities** on the right, choose **Machine learning online endpoint** (3) for **Managed identity**. Finally, under **Selected members**, choose the **llama3-1-8b-nim-endpoint-aml-1** (4), of choice and then continue by clicking on **Select** (5) and **Next** (6).

   ![](../../Coach/media/bash5.png)

1. Click on **Review + assign**.

### Task 11: Create AzureML Deployment of the NIM Container

Create an AzureML deployment with the NIM container obtained from the provided Azure container registry.

1. Run the following command to **create AzureML deployment of the NIM container**.

   ```cmd
   ./5_create_deployment.sh
   ```

### Task 12: Verify Your Connection

1. Return to the **Azure Portal**.

2. Open the **ml-workspace** and click on **Launch studio**.

   ![](../../Coach/media/nvverify1.png)

3. This will take you to **AML Studio**. From the left-hand menu, select **Endpoints** and choose your endpoint.

   ![](../../Coach/media/nvverify2.png)

4. Go to the **Consume** tab, then copy the **REST endpoint** and **Primary key**.

   ![](../../Coach/media/nvverify3.png)

5. In VS Code, open the **`test_chat_completions.sh`** file. Replace the following headers `<your-azureml-endpoint>`, `<your-azureml-endpoint-token>`, and `<deployment-name>` with the appropriate values. Ensure the **deployment-name** matches the one in your `config.sh` file.

   >**Note:** Ensure to add **/v1/chat/completions** towards the end of the endpoint.

6. Once you have updated all the headers, the code should look similar to the following:

   ```
   #!/bin/bash
   curl -X 'POST' \
     'https://llama-3-1-8b-nim-endpoint-aml-1.eastus2.inference.ml.azure.com/v1/chat/completions' \
     -H 'accept: application/json' \
     -H 'azureml-model-deployment: llama3-1-8b-nim-deployment-aml-1' \
     -H 'Authorization: Bearer 3L3s8qb6dCQq7TTgorFnwDVZT8qsvId5' \
     -H 'Content-Type: application/json' \
     -d '{
     "messages": [
       {
         "content": "You are a polite and respectful chatbot helping people plan a vacation.",
         "role": "system"
       },
       {
         "content": "What should I do for a 4 day vacation in Spain?",
         "role": "user"
       }
     ],
     "model": "meta/llama-3.1-8b-instruct",
     "max_tokens": 500,
     "top_p": 1,
     "n": 1,
     "stream": false,
     "stop": "\n",
     "frequency_penalty": 0.0
   }'
   
   ```

1. Run the following command to Verify Connection.

   ```
   ./test_chat_completions.sh
   ```

1. You will see the output similar to the below screenshot.

   ![](../media/llama-output.png)  