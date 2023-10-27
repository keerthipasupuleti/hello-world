# hello-world
just another repository
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app
spec:
  replicas: 3  # Adjust the number of replicas as needed
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
        - name: node-app
          image: harbor.example.com/your-repo/node-app:latest  # Replace with your Harbor repository and image name
          ports:
            - containerPort: 3000


service.yaml file


apiVersion: v1
kind: Service
metadata:
  name: node-app-service
spec:
  selector:
    app: node-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer


gitlab workflow


stages:
  - build
  - deploy

variables:
  DOCKER_REGISTRY: harbor.example.com  # Replace with your Harbor registry URL
  EKS_CLUSTER_NAME: your-eks-cluster-name
  AWS_DEFAULT_REGION: your-region
  AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID  # Define this variable in your GitLab project's CI/CD settings
  AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY  # Define this variable in your GitLab project's CI/CD settings

build:
  stage: build
  script:
    - echo "Building and pushing Docker image to Harbor"
    - docker build -t $DOCKER_REGISTRY/your-repo/node-app:$CI_COMMIT_SHA .
    - docker login -u $DOCKER_REGISTRY_USERNAME -p $DOCKER_REGISTRY_PASSWORD $DOCKER_REGISTRY
    - docker push $DOCKER_REGISTRY/your-repo/node-app:$CI_COMMIT_SHA
  only:
    - master  # Adjust the branch name as needed

deploy:
  stage: deploy
  script:
    - echo "Configuring AWS CLI"
    - aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
    - aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
    - aws eks update-kubeconfig --name $EKS_CLUSTER_NAME --region $AWS_DEFAULT_REGION
    - kubectl apply -f node-app-deployment.yaml
    - kubectl apply -f node-app-service.yaml
  only:
    - master  # Adjust the branch name as needed
