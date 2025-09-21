# Deploy an Azure Kubernetes Service (AKS) cluster using Terraform

## Login to your Azure account

First, log into your Azure account and authenticate using one of the methods described in the following section.

Terraform only supports authenticating to Azure with the Azure CLI. Authenticating using Azure PowerShell isn't supported. Therefore, while you can use the Azure PowerShell module when doing your Terraform work, you first need to [authenticate to Azure](https://learn.microsoft.com/en-us/azure/developer/terraform/authenticate-to-azure).

[](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-terraform?pivots=development-environment-azure-cli#implement-the-terraform-code)
## Implement the Terraform code

1. Create a directory you can use to test the sample Terraform code and make it your current directory.   

## Initialize Terraform

Run [terraform init](https://www.terraform.io/docs/commands/init.html) to initialize the Terraform deployment. This command downloads the Azure provider required to manage your Azure resources.

ConsoleCopy

```
terraform init -upgrade
```

**Key points:**

- The `-upgrade` parameter upgrades the necessary provider plugins to the newest version that complies with the configuration's version constraints.

[](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-terraform?pivots=development-environment-azure-cli#create-a-terraform-execution-plan)

## Create a Terraform execution plan

Run [terraform plan](https://www.terraform.io/docs/commands/plan.html) to create an execution plan.

ConsoleCopy

```
terraform plan -out main.tfplan
```

**Key points:**

- The `terraform plan` command creates an execution plan, but doesn't execute it. Instead, it determines what actions are necessary to create the configuration specified in your configuration files. This pattern allows you to verify whether the execution plan matches your expectations before making any changes to actual resources.
- The optional `-out` parameter allows you to specify an output file for the plan. Using the `-out` parameter ensures that the plan you reviewed is exactly what is applied.

[](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-terraform?pivots=development-environment-azure-cli#apply-a-terraform-execution-plan)

## Apply a Terraform execution plan

Run [terraform apply](https://www.terraform.io/docs/commands/apply.html) to apply the execution plan to your cloud infrastructure.

ConsoleCopy

```
terraform apply main.tfplan
```

**Key points:**

- The example `terraform apply` command assumes you previously ran `terraform plan -out main.tfplan`.
- If you specified a different filename for the `-out` parameter, use that same filename in the call to `terraform apply`.
- If you didn't use the `-out` parameter, call `terraform apply` without any parameters.

[](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-terraform?pivots=development-environment-azure-cli#verify-the-results)

## Verify the results

1. Get the Azure resource group name using the following command.
    
    ConsoleCopy
    
    ```
    resource_group_name=$(terraform output -raw resource_group_name)
    ```
    
2. Display the name of your new Kubernetes cluster using the [az aks list](https://learn.microsoft.com/en-us/cli/azure/aks#az-aks-list) command.
    
    Azure CLICopy
    
    Open Cloud Shell
    
    ```
    az aks list \
      --resource-group $resource_group_name \
      --query "[].{\"K8s cluster name\":name}" \
      --output table
    ```
    
3. Get the Kubernetes configuration from the Terraform state and store it in a file that `kubectl` can read using the following command.
    
    ConsoleCopy
    
    ```
    echo "$(terraform output kube_config)" > ./azurek8s
    ```
    
4. Verify the previous command didn't add an ASCII EOT character using the following command.
    
    ConsoleCopy
    
    ```
    cat ./azurek8s
    ```
    
    **Key points:**
    
    - If you see `<< EOT` at the beginning and `EOT` at the end, remove these characters from the file. Otherwise, you may receive the following error message: `error: error loading config file "./azurek8s": yaml: line 2: mapping values are not allowed in this context`
5. Set an environment variable so `kubectl` can pick up the correct config using the following command.
    
    ConsoleCopy
    
    ```
    export KUBECONFIG=./azurek8s
    ```
    
6. Verify the health of the cluster using the `kubectl get nodes` command.
    
    ConsoleCopy
    
    ```
    kubectl get nodes
    ```
    

**Key points:**

- When you created the AKS cluster, monitoring was enabled to capture health metrics for both the cluster nodes and pods. These health metrics are available in the Azure portal. For more information on container health monitoring, see [Monitor Azure Kubernetes Service health](https://learn.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-overview).
- Several key values classified as output when you applied the Terraform execution plan. For example, the host address, AKS cluster user name, and AKS cluster password are output.

## Deploy the application

To deploy the application, you use a manifest file to create all the objects required to run the [AKS Store application](https://github.com/Azure-Samples/aks-store-demo). A [Kubernetes manifest file](https://learn.microsoft.com/en-us/azure/aks/concepts-clusters-workloads#deployments-and-yaml-manifests) defines a cluster's desired state, such as which container images to run. The manifest includes the following Kubernetes deployments and services:

1. Create a file named `aks-store-quickstart.yaml` and copy in the following manifest.

2. If you create and save the YAML file locally, then you can upload the manifest file to your default directory in CloudShell by selecting the **Upload/Download files** button and selecting the file from your local file system.
    
3. Deploy the application using the `kubectl apply` command and specify the name of your YAML manifest.
    
    ConsoleCopy
    
    ```
    kubectl apply -f ./kubernetes-demo-app/aks-store-quickstart.yaml
    ```
    
    The following example output shows the deployments and services:
    
    OutputCopy
    
    ```
    deployment.apps/rabbitmq created
    service/rabbitmq created
    deployment.apps/order-service created
    service/order-service created
    deployment.apps/product-service created
    service/product-service created
    deployment.apps/store-front created
    service/store-front created
    ```
    

[](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-terraform?pivots=development-environment-azure-cli#test-the-application)

### Test the application

When the application runs, a Kubernetes service exposes the application front end to the internet. This process can take a few minutes to complete.

1. Check the status of the deployed pods using the `kubectl get pods` command. Make all pods are `Running` before proceeding.
    
    ConsoleCopy
    
    ```
    kubectl get pods
    ```
    
2. Check for a public IP address for the store-front application. Monitor progress using the `kubectl get service` command with the `--watch` argument.
    
    Azure CLICopy
    
    Open Cloud Shell
    
    ```
    kubectl get service store-front --watch
    ```
    
    The **EXTERNAL-IP** output for the `store-front` service initially shows as _pending_:
    
    OutputCopy
    
    ```
    NAME          TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
    store-front   LoadBalancer   10.0.100.10   <pending>     80:30025/TCP   4h4m
    ```
    
3. Once the **EXTERNAL-IP** address changes from _pending_ to an actual public IP address, use `CTRL-C` to stop the `kubectl` watch process.
    
    The following example output shows a valid public IP address assigned to the service:
    
    OutputCopy
    
    ```
    NAME          TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)        AGE
    store-front   LoadBalancer   10.0.100.10   20.62.159.19   80:30025/TCP   4h5m
    ```
    
1. Open a web browser to the external IP address of your service to see the Azure Store app in action.

## Clean up resources

[](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-terraform?pivots=development-environment-azure-cli#delete-aks-resources)

### Delete AKS resources

When you no longer need the resources created via Terraform, do the following steps:

1. Run [terraform plan](https://www.terraform.io/docs/commands/plan.html) and specify the `destroy` flag.
    
    ConsoleCopy
    
    ```
    terraform plan -destroy -out main.destroy.tfplan
    ```
    
    **Key points:**
    
    - The `terraform plan` command creates an execution plan, but doesn't execute it. Instead, it determines what actions are necessary to create the configuration specified in your configuration files. This pattern allows you to verify whether the execution plan matches your expectations before making any changes to actual resources.
    - The optional `-out` parameter allows you to specify an output file for the plan. Using the `-out` parameter ensures that the plan you reviewed is exactly what is applied.
2. Run [terraform apply](https://www.terraform.io/docs/commands/apply.html) to apply the execution plan.
    
    ConsoleCopy
    
    ```
    terraform apply main.destroy.tfplan
    ```
    

[](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-terraform?pivots=development-environment-azure-cli#delete-service-principal)

### Delete service principal

1. Get the service principal ID using the following command.
    
    Azure CLICopy
    
    Open Cloud Shell
    
    ```
    sp=$(terraform output -raw sp)
    ```
    
2. Delete the service principal using the [az ad sp delete](https://learn.microsoft.com/en-us/cli/azure/ad/sp#az-ad-sp-delete) command.
    
    Azure CLICopy
    
    Open Cloud Shell
    
    ```
    az ad sp delete --id $sp
    ```

