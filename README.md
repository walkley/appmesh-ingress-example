## 1. create eks cluster
```bash
#REGION=$(curl -sS http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .region)
REGION=ap-southeast-1
MESH_NAME=color-mesh
eksctl create cluster --region $REGION --name appmesh-test --version 1.13 --appmesh-access
```

## 2. install appmesh controller
```bash
kubectl apply -f https://raw.githubusercontent.com/aws/aws-app-mesh-controller-for-k8s/master/deploy/all.yaml
```

## optional, check resources
```bash
kubectl rollout status deployment app-mesh-controller -n appmesh-system
kubectl get crd
```

## 3. install appmesh side car injector
```bash
curl https://raw.githubusercontent.com/aws/aws-app-mesh-inject/master/scripts/install.sh | bash
```

## 4. install color teller sample application
```bash
kubectl apply -f https://raw.githubusercontent.com/aws/aws-app-mesh-controller-for-k8s/v0.1.0/examples/color.yaml
```

## optional, test
```bash
kubectl run -n appmesh-demo -it curler --image=tutum/curl /bin/bash
```

## 5. install ingress-nginx and app mesh virtual node
```bash
kubectl create ns ingress-nginx
kubectl label namespace ingress-nginx appmesh.k8s.aws/sidecarInjectorWebhook=enabled
kubectl apply -f ./nginx-ingress-appmesh.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/aws/service-nlb.yaml
kubectl apply -f ./nginx-ingress.yaml
```

## optional, watch query result of ingress
```bash
ELB_URL=$(kubectl get svc ingress-nginx -n ingress-nginx -o jsonpath="{.status.loadBalancer.ingress[0].hostname}")
watch -t curl -s http://$ELB_URL/color; echo;
```

## 6. clean up
```bash
kubectl delete ns ingress-nginx
kubectl delete crd meshes.appmesh.k8s.aws
kubectl delete crd virtualnodes.appmesh.k8s.aws
kubectl delete crd virtualservices.appmesh.k8s.aws
kubectl delete namespace appmesh-system
kubectl delete namespace appmesh-inject
```
