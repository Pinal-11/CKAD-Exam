- Kustomize comes built-in with kubectl so no other package needs to bs installed.


Commands:

kustomize version --short
kustomize build k8s/ --> provide the whole folder

folder looks likke
k8s
|-- nginx-deploy.yaml
|-- nginx-service.yaml
|-- kustomization.yaml   --> This is mandatory and it should be the same name 

kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta  # this feild is optional you can define it. ot if you not define it then it will fetch automatically
kind: Kustomization     # This is the same as the apiVersion

# kubernetes resources to be managed by kustomize
resources:
    nginx-deploy.yaml
    nginx-service.yaml

# Customization that need to be made
commonLabels:
    company: anything

- Kustomize looks for a kustomization file which contains:
    List of all the k8s manifest kustomize should be manage
    All of the customiations that should be applied

- The 'kustomize build k8s/' command combined all the manifest and applies the defined transformations.
- The 'kustomize build' command does not apply/deploy the k8s resources to a cluster
    The output needs to redirected to the 'kubectl apply' command

# The below ways we can apply the kustomizarion
1st way -> kustomize build k8s/ | kubectl apply -f -  
2nd way -> kubectl apply -k k8s/

## To delete it
1st way -> kustomize build k8s/ | kubectl delete -f - 
2nd way -> kubect delere -k k8s/

![alt text](<Screenshot 2026-08-19 at 7.21.40 PM.png>)

![alt text](<Screenshot 2026-08-19 at 7.21.40 PM-1.png>)

![alt text](<Screenshot 2026-08-19 at 7.26.57 PM.png>)


# Common Transformation
- commonLabel - adds a label to all k8s resources
![alt text](<Screenshot 2026-08-20 at 12.13.51 PM.png>)

- namePrefix/Suffix - adds a common prefix-suffix to all resource names
![alt text](<Screenshot 2026-08-20 at 12.16.04 PM.png>)

- Namespaces - adds a common nd to all resources
![alt text](image.png)

- commonAnnotations - 
![alt text](<Screenshot 2026-08-20 at 12.17.06 PM.png>)

# Image Transformer
1. 
![alt text](<Screenshot 2026-08-20 at 12.19.27 PM.png>)

2. 
![alt text](<Screenshot 2026-08-20 at 12.21.51 PM-1.png>)



# Patches
- Kustomize patches provide another method to modifying kubernetes configs
- Patches provides a more surgical approch to targeting one or more specific section in kubernetes resources
- To create a patch 3 parameter must be provided:
    - Operation Type: add/remove/replace
    - Target: In which resource patch apply
        - kind
        - Version/Group
        - Name
        - Namespace
        - labelSelector
        - AnnotationSelector
    - Value: What is the value that will either be replaced or added with. (only need for add/replace operations)
    
    ![alt text](image-3.png)

    ![alt text](image-1.png)

    ![alt text](image-2.png)

    ![alt text](image-4.png) 
    
    ![alt text](image-5.png)

    ![alt text](image-6.png)

    ![alt text](image-7.png)

    ![alt text](image-8.png)

    ![alt text](image-9.png)


# Overlays
![alt text](image-10.png)

![alt text](image-11.png)

![alt text](image-12.png)

# Components

![alt text](image-13.png)