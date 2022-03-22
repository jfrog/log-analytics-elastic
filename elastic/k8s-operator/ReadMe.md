https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-eck.html
kubectl apply -f https://download.elastic.co/downloads/eck/2.1.0/operator.yaml

PASSWORD=$(kubectl get secret -n elastic elasticsearch-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')

kubectl get secret elasticsearch-es-elastic-user -n elastic -o=jsonpath='{.data.elastic}' | base64 --decode; echo
