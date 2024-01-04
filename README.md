
# Project Title

Carrental Project

## The Purpose of the project

This project is a Carrental project created using kubernetes,docker,jenkins,terraform.

## İnformation about the project

First, an image was created for backend and frontend containers. A Docker-compose file was prepared for these containers to work, and this Docker-compose file was executed. Thus, the code was tested to see if it was running. To include Kubernetes in the project, the docker-compose file was converted to the yaml file format that Kubernetes wants using the Kompose tool. These files were also customized using the Kustomize tool. The Kubernetes environment was created using AWS's Eks tool. An Ingress controller was used for load balancing of the containers. All of these processes were automated through Jenkins (compilation of Dockerfile, sending to AWS Ecr, pulling from there and creating containers, creating Eks, etc.). Finally, monitoring and control were done using AWS Prometheus and Grafana.


## Used technologies

**Frontend:** React

**Backend:** Java Spring Boot

**Database:** Postgresql
 
**infrastructure:** 
Kubernetes-Docker-Jenkins-Terraform-Kompose-Kustomize-Aws Prometheus-Aws Grafana - Aws Eks-Aws Ecr
  

## Screenshots

![Uygulama Ekran Görüntüsü](https://github.com/kezvur/final_project_bluerentalcar/blob/main/bluerental.JPG)


## Feedback

vural93keziban@gmail.com

keziban@kezibanvural.com
  
