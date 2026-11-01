# CCA 2: CI/CD Pipeline for Doctor Appointment System

**GitHub repository:** https://github.com/krishmadhvani1164-pixel/doctor-appoitment
**Docker Hub image:** krissh1704/doctor-appointment

## 1. Project chosen
Doctor Appointment System: a small Flask + SQLite web application. A patient can book an appointment with a doctor for a date and time slot, double booking is blocked, and appointments can be cancelled. It has a `/health` endpoint used by Kubernetes. The project was built for this assignment and kept small so the focus stays on automation.

## 2. Repository contents
- `app.py`, `templates/`, `static/`, `requirements.txt`: application source
- `test_app.py`: pytest unit tests (5 tests)
- `Dockerfile`: builds the container image
- `Jenkinsfile`: declarative pipeline
- `k8s/deployment.yaml`, `k8s/service.yaml`: Kubernetes manifests

## 3. Pipeline stages (Jenkinsfile)
1. **Checkout**: Jenkins pulls the latest code from GitHub.
2. **Build and Test**: installs dependencies in a Python container and runs `pytest`.
3. **Docker Build**: builds the image from the Dockerfile, tagged with the Jenkins build number and `latest`.
4. **Push Image**: logs in to Docker Hub using credentials stored in Jenkins (`dockerhub-creds`) and pushes both tags.
5. **Deploy to Minikube**: replaces the image placeholder in `deployment.yaml` with the new tag and applies the Deployment and Service with `kubectl`, then waits for the rollout.
6. **Verification**: runs `kubectl get pods`, `kubectl get deployment` and `kubectl get svc`.

## 4. Jenkins setup on AWS
- Launched an EC2 instance (Ubuntu, t3.medium, 25 GB disk).
- Security group allows ports 22 (SSH), 8080 (Jenkins) and 5000 (app).
- Installed Docker, Java, Jenkins, kubectl and Minikube (Docker driver) on the instance.
- Installed the Docker Pipeline plugin and added Docker Hub credentials (username and access token) in Jenkins Credentials.
- Created a Pipeline job using "Pipeline script from SCM" pointing to the GitHub repository and the `Jenkinsfile`.
- Gave the `jenkins` user access to Docker and the Minikube kubeconfig.

## 5. How the image was pushed
The Jenkins pipeline logs in to Docker Hub with the stored credentials (no password in code) and pushes `krissh1704/doctor-appointment:<build-number>` and `:latest`.

## 6. Deployment and access
- Deployment runs 2 replicas, with a readiness probe on `/health`. The Service is of type NodePort.
- The image is pulled from Docker Hub by the Minikube pods.
- Because Minikube runs inside Docker on the EC2 instance, the app is exposed with:
  `kubectl port-forward --address 0.0.0.0 svc/doctor-appointment 5000:5000`
- Open `http://<EC2-public-IP>:5000` in a browser.

## 7. Change for the second run
The header and footer text was changed from **Version 1** to **Version 2** in `templates/index.html` and pushed to GitHub. Jenkins was run again, a new image tag was pushed, the Deployment was updated, and the browser then showed **Version 2**.

## 8. Screenshots
Add screenshots here: Jenkins stage view (run 1 and run 2), Docker Hub tags, `kubectl get pods/deployment/svc`, and the app in the browser (Version 1 and Version 2).
