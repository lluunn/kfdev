# Example for doing TF Serving in Kubeflow

## Steps for using TF Serving with IAP + Istio
1. Deploy Kubeflow with kfctl. See [doc](https://www.kubeflow.org/docs/gke/deploy/deploy-cli/).
   Choose IAP option, so need to create oauth client.
   In 0.5 release, Istio is not turned on, so please build kfctl with master branch.

2. Make sure the Kubeflow deployment is ready. https://{KFAPP}.endpoints.{PROJECT}.cloud.goog endpoint should be accessible.

3. Modify the manifest `serving-istio.yaml` as needed. For example, the model GCS location at line 22, model name at line 21.
4. `kubectl apply -f serving-istio.yaml`
5. Send prediction request. See [instructions](https://github.com/kubeflow/kubeflow/blob/master/docs/gke/tf_serving_iap.md). Basically, create a service account with proper IAM, set GOOGLE_APPLICATION_CREDENTIALS pointing to the service account key, and then do `python iap_request.py https://HOST/tfserving/models/mnist --input=INPUT_FILE $CLIENT_ID`

### Prediction using port-forward
1. Find the pod name using `kubectl get pods -n kubeflow`
2. `kubectl port-forward -n kubeflow POD_NAME 8500`
3. curl -X POST -d @INPUT_FILE  localhost:8500/v1/models/mnist:predict

## Notes
- For testing model, we can use mnist model at gs://kubeflow-examples-data/mnist and input file [here](https://github.com/kubeflow/kubeflow/blob/master/components/k8s-model-server/test-data/mnist_input.json)