# Webapp Kubernetes Deployment

## Secret Management

## Repository Structure

webapp-k8s/
├── base/                          
│   ├── deployment.yaml
│   ├── kustomization.yaml            
│   └── service.yaml   
│
└── overlays/
    ├── production/                  
    │   ├── kustomization.yaml
    │   ├── namespace.yaml         
    │   ├── replica_patch.yaml
    │
    └── staging/        
        ├── kustomization.yaml
        ├── namespace.yaml     
        └── replica_patch.yaml


**Chosen approach for this exercise:** Kustomize `secretGenerator` with literals.

## Why I Chose This Approach

| Reason                    |  Explanation                         
|---------------------------|--------------------------------------
| No Secret YAML in Git     | Secrets are generated dynamically at apply time, never stored as YAML files
| Built into Kustomize      | No additional tools, controllers, or CRDs required
| Environment-Specific      | Different credentials per environment (staging vs production)
| GitOps Compatible         | Works with standard kubectl apply -k commands
| Simple & Auditable        | Clear what credentials each environment uses


## How It Works
1. Kustomize reads the secretGenerator block at build time
2. Creates a Kubernetes Secret object in memory 
3. Automatically base64 encodes the literal values
4. Applies the Secret to the cluster alongside other resources
5. No secret material ever touches your filesystem


**Why:** Simple, built-in, and avoids committing raw Secret YAML. For a real public repo, I would:
1. Use `secretGenerator` with `files` and `.gitignore` the actual files, OR
2. Use SealedSecrets (encrypts secrets into a CRD safe for Git), OR
3. Use External Secrets Operator (fetches from AWS Secrets Manager).

**Trade-off:** Literals in `kustomization.yaml` are still plain text. In production, use sealed-secrets or external secrets.

## Deploy

```bash
# Staging (1 replica, nginx:1.25)
kubectl apply -k overlays/staging

# Production (3 replicas, nginx:1.27)
kubectl apply -k overlays/production