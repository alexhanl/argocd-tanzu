# argocd-tanzu

openssl req -new -newkey rsa:4096 -nodes -keyout argocd-temp.key -out argocd-temp.csr -subj "/CN=argocd-temp"

<!-- cat argocd-temp.csr | base64 | tr -d '\n' -->

cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: argocd-temp-csr
spec:
  request: $(cat argocd-temp.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  # expirationSeconds: 86400  # one day
  usages:
  - client auth
EOF

kubectl get csr

kubectl certificate approve argocd-temp-csr

kubectl get csr argocd-temp-csr -o jsonpath='{.status.certificate}' | base64 --decode > argocd-temp-csr.crt

kubectl config view -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' --raw | base64 --decode - > kubernetes-ca.crt

kubectl config set-cluster $(kubectl config view -o jsonpath='{.clusters[0].name}') --server=$(kubectl config view -o jsonpath='{.clusters[0].cluster.server}') --certificate-authority=kubernetes-ca.crt --kubeconfig=argocd-temp-config --embed-certs

kubectl config set-credentials argocd-temp --client-certificate=argocd-temp-csr.crt --client-key=argocd-temp.key --embed-certs --kubeconfig=argocd-temp-config

kubectl config set-context argocd-temp --cluster=$(kubectl config view -o jsonpath='{.clusters[0].name}') --namespace=default --user=argocd-temp --kubeconfig=argocd-temp-config



kubectl config use-context argocd-temp --kubeconfig=argocd-temp-config

kubectl get po  --kubeconfig=argocd-temp-config


cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: argocd-temp-clusterrolebinding
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: argocd-temp
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: ""
EOF


argocd app create tkgs-prod --repo https://github.com/alexhanl/argocd-tanzu.git --path tkgs --dest-server https://192.168.110.101:6443 --dest-namespace prod




https://www.infracloud.io/blogs/multicluster-gitops-argocd/
https://argo-cd.readthedocs.io/en/stable/getting_started/
https://medium.com/swlh/how-we-effectively-managed-access-to-our-kubernetes-cluster-38821cf24d57
