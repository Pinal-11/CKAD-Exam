Helm -- Helm is use full when we want to install the whole bunch of object whitout worring about the individual yaml files
It work as the package manager

Helm commands
helm install <package-name>
helm install wordpress
helm upgrade wordpress
helm rollback wordpress
helm uninstall wordpress

helm search hub wordpress

helm repo add bitnami https://charts.bitnami.com/bitnami

helm search repo wordpress

helm repo list

helm install <release-name> <chart-name>
helm install release1 bitnami/wordpress
helm install release2 bitnami/wordpress
helm install release3 bitnami/wordpress


helm list

helm uninstall my-release

helm pull --untar bitnami/wordpress

ls wordpress

helm install release4 ./wordpress
