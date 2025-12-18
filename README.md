# Tea House

This is the Tea House landing page project.

## Project Structure

- `app/`: Application source code (HTML, CSS, images)
- `Dockerfile`: Docker configuration for containerizing the app
- `Jenkinsfile`: CI/CD pipeline for building, scanning, and deploying
- `k8s/`: Kubernetes deployment and service files
- `sonar-project.properties`: SonarQube configuration for code analysis
- `package.json`: Node.js package file (minimal for static site)

## Deployment

1. Push code to repository.
2. Jenkins will trigger the pipeline:
   - Build Docker image
   - Run SonarQube scan
   - Push image to Nexus
   - Deploy to Kubernetes

## Credentials

- SonarQube: URL http://sonarqube.imcc.com/, Token provided in sonar-project.properties
- Nexus: URL http://nexus.imcc.com/, Username: student, Password: Imcc@2025
- Jenkins: URL http://jenkins.imcc.com/, Username: student, Password: Changeme@2025

## Local Development

Run `npm start` to serve locally, or use Docker: `docker build -t tea-house . && docker run -p 80:80 tea-house`