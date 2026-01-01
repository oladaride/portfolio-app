**End-to-End Deployment of a Work-in-Progress Portfolio Application Using WSL, Git, Docker, and GitHub Actions**

*Overview*
This documentation describes the complete process of deploying a work-in-progress portfolio application, starting from local development in WSL (Ubuntu) to a live deployment on an EC2 instance, using Git, Docker, and GitHub Actions for automation.

My goal is to demonstrate a full CI/CD pipeline, not application completeness.

The original Windows file path:
`C:\Users\LENOVO\Desktop\portfolio.js`
was converted to a WSL-compatible path:
`/mnt/c/Users/LENOVO/Desktop/portfolio.js`

This path conversion allows WSL to access files stored on the Windows filesystem.

## Git Version Control Setup

Inside the project directory, Git was initialized:
`git init`
The repository status was checked:
`git status`
All project files were staged:
`git add .`
The initial commit was created:
`git commit -m "first commit"`
The default branch was renamed from master to main to follow modern Git conventions:
`git branch -M main`

## GitHub Repository Configuration

A new repository was created on GitHub.
The remote repository was added locally:

`git remote add origin https://github.com/oladaride/portfolio-app.git`
The project was pushed to GitHub:
`git push -u origin main`

## Containerization with Docker

To containerize the application, a Dockerfile was created in the project root.

```
FROM nginx:alpine
COPY . /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
Explanation:
FROM nginx:alpine
Uses a lightweight Nginx base image.
COPY . /usr/share/nginx/html
Copies the portfolio files into the directory Nginx serves from.
EXPOSE 80
Exposes port 80 for HTTP traffic.
CMD
Starts Nginx in the foreground so the container remains running.

The Docker image was built locally:

`sudo docker build -t portfolio-app .`
This produced a Docker image named:

portfolio-app:latest

The container was started:

`sudo docker run -d -p 80:80 --name portfolio portfolio-app`
Command Breakdown:
-d → Runs the container in detached (background) mode
-p 80:80 → Maps the host’s port 80 to the container’s port 80
--name portfolio → Assigns a name to the container
portfolio-app → The image being used

The application became accessible by visiting the EC2 instance’s public IP address in a browser.

## CI/CD Automation with GitHub Actions

To automate deployment, GitHub Actions was configured.
GitHub Secrets Configuration
Repository secrets were added under:
Settings → Secrets and variables → Actions:
EC2 Secrets
EC2_HOST
EC2_SSH_KEY
Docker Hub Secrets
DOCKER_USERNAME
DOCKER_PASSWORD

## GitHub Actions Workflow

A workflow file was created:
`.github/workflows/deploy.yml`
Workflow configuration:
```
name: Deploy Portfolio App

on:
  push:
    branches: [ "main" ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-22.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Build Docker image
        run: docker build -t portfolio-app .

      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin

      - name: Push image to Docker Hub
        run: |
          docker tag portfolio-app ${{ secrets.DOCKER_USERNAME }}/portfolio-app:latest
          docker push ${{ secrets.DOCKER_USERNAME }}/portfolio-app:latest

      - name: Deploy to EC2
        uses: appleboy/ssh-action@v0.1.10
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull ${{ secrets.DOCKER_USERNAME }}/portfolio-app:latest
            docker stop portfolio || true
            docker rm portfolio || true
            docker run -d -p 80:80 --name portfolio ${{ secrets.DOCKER_USERNAME }}/portfolio-app:latest
```
## Deployment Flow Summary
Code is pushed to the main branch.
GitHub Actions builds a Docker image.
The image is pushed to Docker Hub.
The EC2 instance pulls the latest image.
The existing container is stopped and removed.
A new container is started with the updated image.

## Conclusion

This setup demonstrates a complete end-to-end CI/CD pipeline for a portfolio application using:
WSL for local development
Git and GitHub for version control
Docker for containerization
GitHub Actions for automated deployment
EC2 as the hosting environment

Although the application itself is incomplete, the deployment pipeline is fully functional and production-oriented, and that's one of the core responsibilities of a DevOps Engineer.
























