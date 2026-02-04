Kagent + Ollama on Kubernetes (kind / WSL)
This repository demonstrates how to run kagent with Ollama as the LLM provider entirely inside Kubernetes using free, local LLaMA models.

This setup is suitable for kind clusters running on WSL and avoids networking issues by keeping all components in-cluster.


Prerequisites
Kubernetes cluster (kind recommended)
kubectl
Helm
Adequate CPU and memory (local LLMs are resource intensive)


Step 1: Deploy Ollama and LLaMA Model
kubectl create ns ollama
kubectl apply -f   ollama-model.yaml

Note: Increase CPU and memory limits in ollama-model.yaml for better performance.

Credit: https://www.cloudnativedeepdive.com/deploying-local-ai-agents-in-kubernetes/

Verify Ollama is running: (might take some time)
kubectl get pods -n ollama

Confirm the Model
kubectl exec -n ollama deployment/ollama -- ollama list



Step 2: Install kagent CRDs
kagent requires CRDs before installing the main Helm chart.

helm install kagent-crds oci://ghcr.io/kagent-dev/kagent/helm/kagent-crds \
  --namespace kagent \
  --create-namespace

Reference documentation: https://kagent.dev/docs/kagent/introduction/installation



Step 3: Install kagent with Ollama Provider
Install or upgrade kagent and configure Ollama as the default LLM provider:

helm upgrade --install kagent oci://ghcr.io/kagent-dev/kagent/helm/kagent \
  --namespace kagent \
  --set providers.default=ollama \
  --set providers.ollama.provider=Ollama \
  --set providers.ollama.config.host=http://ollama.ollama.svc.cluster.local:11434 \
  --set providers.ollama.model=llama3.2

This configuration:
Sets Ollama as the default provider
Uses the in-cluster Ollama service
Runs the free llama3.2 model

Check if the pods are ready
kubectl get pods -n kagents

Step 4: Port-forward the kagent UI
Expose the kagent UI locally:
kubectl port-forward -n kagent svc/kagent-ui 8080:8080


Step 5: Access the UI
Open your browser and navigate to:
http://localhost:8080/



Notes
No external API keys required
Fully local and offline after model download
Ideal for learning, demos, and experimentation


Cleanup
To remove all components:

helm uninstall kagent -n kagent
helm uninstall kagent-crds -n kagent
kubectl delete namespace kagent
kubectl delete -f ollama-model.yaml
