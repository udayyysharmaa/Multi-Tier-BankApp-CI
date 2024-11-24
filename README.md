# Update kubeconfig file

```
aws eks --region ap-south-1 update-kubeconfig --name devopsshack-cluster
```

# Dowload Argo cd File

'''
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
'''
