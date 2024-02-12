# Step 4: Test KServe

Testing KServe involves deploying a simple inference service and querying it. Follow the steps below:

## Create a Namespace for Testing

```bash
kubectl create namespace kserve-test
```

# Deploy an InferenceService
Deploy the SKLearn Iris model as an inference service:
```bash
kubectl apply -n kserve-test -f - <<EOF
apiVersion: "serving.kserve.io/v1beta1"
kind: "InferenceService"
metadata:
  name: "sklearn-iris"
spec:
  predictor:
    sklearn:
      storageUri: "gs://kfserving-examples/models/sklearn/1.0/model"
EOF
```

# Determine Ingress Host and Port
Retrieve the IP address and port of the Istio ingress gateway:

```bash
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'); echo $INGRESS_HOST
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}'); echo $INGRESS_PORT
export SERVICE_HOSTNAME=$(kubectl get inferenceservice sklearn-iris -n kserve-test -o jsonpath='{.status.url}' | cut -d "/" -f 3); echo $SERVICE_HOSTNAME
```

# Prepare the Input Data
Create a JSON file named iris-input.json with input data:
```bash
{
  "instances": [
    [6.8,  2.8,  4.8,  1.4],
    [6.0,  3.4,  4.5,  1.6]
  ]
}
```

# Query the InferenceService
Send a request to the inference service:
```bash
curl -v -H "Host: ${SERVICE_HOSTNAME}" -H "Content-Type: application/json" "http://${INGRESS_HOST}:${INGRESS_PORT}/v1/models/sklearn-iris:predict" -d @./iris-input.json
```

# Clean Up
After testing, delete the test namespace:
```bash
kubectl delete ns kserve-test
```

Note: The exact values for INGRESS_HOST and INGRESS_PORT will depend on your environment. Ensure Istio is correctly installed and that the Istio ingress gateway service is exposed externally, which might require specific cloud provider or environment configurations.

By following these steps, you have deployed a KServe InferenceService, tested it with a sample input, and verified that the service is functioning as expected.