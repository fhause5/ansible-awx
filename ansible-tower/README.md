### Install

> https://computingforgeeks.com/how-to-install-ansible-awx-on-ubuntu-linux/?expand_article=1

```
git clone https://github.com/ansible/awx-operator.git
cd awx-operator/
git checkout 2.4.0

kind create cluster --config kind-config.yaml


export NAMESPACE=awx
kubectl create ns ${NAMESPACE}
make deploy

kubectl -n awx get pods

cat <<EOF | kubectl create -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: static-data-pvc
  namespace: awx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
EOF

export SA=awx
kubectl create clusterrolebinding ${SA}-binding --clusterrole=cluster-admin --serviceaccount=aws:${SA}

cat <<EOF | kubectl -n awx create -f -
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  service_type: nodeport
  projects_persistence: true
  projects_storage_access_mode: ReadWriteOnce
  web_extra_volume_mounts: |
    - name: static-data
      mountPath: /var/lib/projects
  extra_volumes: |
    - name: static-data
      persistentVolumeClaim:
        claimName: static-data-pvc
EOF
kubectl -n awx port-forward service/awx-service 32383:80
kubectl -n awx get secret awx-admin-password -o jsonpath="{.data.password}" | base64 --decode

```