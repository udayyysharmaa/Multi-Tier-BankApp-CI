# Update kubeconfig file

```
aws eks --region ap-south-1 update-kubeconfig --name devopsshack-cluster
```

# Dowload Argo cd File

'''
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
'''

# Password Argo cd 

```
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```
