# CKAD-Exam

Commands:
k create -f deployment-defination.yml
k get deploy
k apply -f deploy-def.yaml
k set image deploy <deployment-name> nginx=nginx:1.9.2
k rollout status deploy <deployment-name>
k rollout history deploy <deployment-name>
k rollout undo deploy <deployment-name>