
# Build version without Ollama for multi-platform 
docker buildx build --build-arg USE_OLLAMA=false --platform linux/amd64,linux/arm64 -t ghcr.io/henn-dt/open-webui:rag-debug .


# Build version with Ollama
docker build --build-arg USE_OLLAMA=true --platform linux/amd64,linux/arm64 -t henn-dt/open-webui:rag-debug-with-ollama .

    First, log in to GitHub Container Registry. You'll need a Personal Access Token (PAT) from GitHub: 
        Go to GitHub.com -> Settings -> Developer Settings -> Personal Access Token
        Create a new token with write:packages and read:packages permissions
        Save the token somewhere safe
         

    Log in to GHCR using your token: 
    echo "YOUR_PAT" | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin


Tag your images following GHCR naming convention:

# For the no-ollama version
docker tag henn-dt/open-webui:rag-debug-no-ollama ghcr.io/henn-dt/open-webui:rag-debug

# For the with-ollama version
docker tag henn-dt/open-webui:rag-debug-with-ollama ghcr.io/henn-dt/open-webui:rag-debug-with-ollama

Push the images:

docker push ghcr.io/henn-dt/open-webui:rag-debug
docker push ghcr.io/henn-dt/open-webui:rag-debug-with-ollama

on Kubernetes:
# Check if you're logged into GHCR
docker login ghcr.io -u YOUR_GITHUB_USERNAME
# Check exact image name exists
docker images | grep open-webui

# for private image create a secret with GitHub credentials:
kubectl create secret docker-registry ghcr-login-secret -n <namespace> \
  --docker-server=ghcr.io \
  --docker-username=YOUR_GITHUB_USERNAME \
  --docker-password=YOUR_GITHUB_PAT \
  --docker-email=YOUR_GITHUB_EMAIL

  Then in your deployment YAML, add the imagePullSecrets reference:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: your-deployment
spec:
  template:
    spec:
      imagePullSecrets:
      - name: ghcr-login-secret     # This should match the secret name you created
      containers:
      - name: your-container
        image: ghcr.io/henn-dt/open-webui:rag-debug