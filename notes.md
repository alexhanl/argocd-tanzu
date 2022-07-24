
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: argocd-admin-clusterrolebinding
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: argocd-admin
roleRef:
  kind: ClusterRole
  name: tkc-manager
  apiGroup: ""
EOF


cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: tkc-manager
rules:
- apiGroups: ["run.tanzu.vmware.com"]
  resources: ["tanzukubernetesclusters"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"] 
EOF