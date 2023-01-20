# otel-operator-azure-monitor

## **Description**
This project is intended to test the opentelemetry operator using auto-instrumentation and exporting metrics to azure app insights.

# Table of Contents
1. [Steps to configure the operator and auto-instrumentation](#configuration)
2. [References](#references)

### **Steps to configure the operator and auto-instrumentation** <a name="configuration"></a>


1. Install certmanager:
    ```
    # kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
    ```
2. Install the opentelemetry operator:

    ```
    # helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts

    # helm repo update

    # helm install --namespace opentelemetry-operator-system opentelemetry-operator open-telemetry/opentelemetry-operator --create-namespace
    ```
3. Create an opentelemetry collector object in kubernetes
    
    **NOTE**: Replace the <INSTRUMENTATION_KEY> placeholder with your application insights instrumentation key that can be retrieved from azure portal:

    ![Alt text](images/instrumentation_key.png "App insights instrumentation key")


    ```
    # kubectl apply -f azuremonitor-otel-collector.yml
    ```
4. Create the autoinstrumentation configuration

    ```
    # kubectl apply -f azuremonitor-autoinstrumentation.yml
    ```
5. Run the deployments with the opentelemetry annotation(In this example, we are using python):

    ```
    # kubectl apply -f example/app/
    ```
6. Port forward the application and make an api call:

    ```
    # kubectl port-forward deployment.apps/client-api 8080:8000

    # curl --location --request POST 'localhost:8080/create/cart' \
        --header 'Content-Type: application/json' \
        --data-raw '{
            "email": "myemail@email.com",
            "products": ["The last of us part I"]
        }'

    # curl --location --request GET 'localhost:8080/get/carts?email=myemail@email.com'
    ```
7. See the telemetry in azure application insights:

![Alt text](images/app_insights_telemetry.png "App insights transaction")

### References <a name="references"></a>

https://github.com/open-telemetry/opentelemetry-helm-charts/tree/main/charts/opentelemetry-operator

https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/azuremonitorexporter

https://medium.com/opentelemetry/using-opentelemetry-auto-instrumentation-agents-in-kubernetes-869ec0f42377

https://github.com/open-telemetry/opentelemetry-operator