# my-app-repo
Automating Docker Image Deployment to Amazon ECR
create the repository in github "my-app-repo
clone the repo in the terminal 
create dockerfile using touch docker - then nano dockerfile- input the configuration
create other dependencies and configurations for the docker container.

docker build -t my-app-repo .
docker run -p 3000:3000 my-app-repo

open the AWS console and create a ECR and push to the AWS

push in the different environment 
dev, staging and Prod

keh@Kenny:~/my-app-repo$ ls
Dockerfile  README.md  index.js  package.json
keh@Kenny:~/my-app-repo$ git switch -c "action branch"
keh@Kenny:~/my-app-repo$ git switch -c action-branch
Switched to a new branch 'action-branch'
keh@Kenny:~/my-app-repo$ git status
On branch action-branch
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        Dockerfile
        index.js
        package.json

nothing added to commit but untracked files present (use "git add" to track)
keh@Kenny:~/my-app-repo$ git add .
keh@Kenny:~/my-app-repo$ git commit -m "docker first commit"
[action-branch fde173f] docker first commit
 3 files changed, 45 insertions(+)
 create mode 100644 Dockerfile
 create mode 100644 index.js
 create mode 100644 package.json
keh@Kenny:~/my-app-repo$ git push

Add GitHub Secrets
Go to:
Settings → Secrets → Actions
Add:
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
These come from an IAM user with ECR permissions.



Create GitHub Actions Workflow

Create file:

.github/workflows/ci.yml
Basic Workflow Structure
name: CI/CD Pipeline

on:
  pull_request:
    branches:
      - dev
      - staging
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          region: ${{ secrets.AWS_REGION }}

      - name: Login to ECR
        run: |
          aws ecr get-login-password --region $AWS_REGION \
          | docker login --username AWS --password-stdin <repo-uri>

      - name: Set environment tag
        id: vars
        run: |
          if [[ "${{ github.base_ref }}" == "dev" ]]; then
            echo "TAG=dev" >> $GITHUB_OUTPUT
          elif [[ "${{ github.base_ref }}" == "staging" ]]; then
            echo "TAG=staging" >> $GITHUB_OUTPUT
          else
            echo "TAG=prod" >> $GITHUB_OUTPUT
          fi

      - name: Build Docker image
        run: docker build -t my-app:${{ steps.vars.outputs.TAG }} .

      - name: Tag image for ECR
        run: |
          docker tag my-app:${{ steps.vars.outputs.TAG }} \
          <repo-uri>:${{ steps.vars.outputs.TAG }}

      - name: Push to ECR
        run: |
          docker push <repo-uri>:${{ steps.vars.outputs.TAG }}


 
