# LAB : Console and Cloud Shell

## Objectives :

In this lab, you learn how to perform the following tasks:

- Get access to Google Cloud.

- Create a Cloud Storage bucket using the Cloud Console.

- Create a Cloud Storage bucket using Cloud Shell.

- Become familiar with Cloud Shell features.

## Steps :

1. Create a bucket using the Cloud Console

  `gsutil mb gs://nwani1`

2. Create a bucket using Cloud Shell  
  
   `gsutil mb gs://nwani2`

3: Explore more Cloud Shell features, UPLOAD A FILE
   We make a file using the Cloud Shell and upload it to our Cloud Bucket.
   
   Run : 
   `nano me.md` 
    
   Insert the following:
   <p> My Name is Victory Nwani </p>

   Hit CTRL + O to save

   Run the following the Cloud Shell to copy the new file into our bucket : 

   `gsutil cp me.md gs://nwani2` 

4. Create a persistent state in Cloud Shell
   - Create and verify an environment variable

    Run the following to create an environment variable :
    `INFRACLASS_REGION=us-central-1`

    Create another environment variable to store PROJECT_ID: 
    `INFRACLASS_PROJECT_ID=$DEVSHELL_ID`

   - Append the environment variable to a file

   Create a file in a new directory

    `nano infraclass/config`

    Append the following lines to the new file:
    `echo INFRACLASS_REGION=$INFRACLASS_REGION >> ~/infraclass/config`
    `echo INFRACLASS_PROJECT_ID=$INFRACLASS_PROJECT_ID >> ~/infraclass/config`

    Save and exit nano  by using `CTRL + O` && `CTRL + X`

    Apply and test variables by using `source` and echoing them: 
    `source infraclass/config`
    `echo $INFRACLASS_PROJECT_ID`

5. Modify the bash profile and create persistence
    - Open and append new line to `.profile`
      `nano .profile`

      Insert at the end :
      `source infraclass/config`

      Save `CTRL + O` and exit `CTRL + X`

6. Exit Console:
   `exit`


# LAB : App Dev - Deploying the Application into Kubernetes Engine: Node.js

## Objectives
In this lab, you learn how to perform the following tasks:

 - Create Dockerfiles to package up the Quiz application frontend and backend code for deployment.
 
 - Harness Cloud Build to produce Docker images.
 
 - Provision a Kubernetes Engine cluster to host the Quiz application.
 
 - Employ Kubernetes deployments to provision replicated Pods into Kubernetes Engine.
 
 - Leverage a Kubernetes service to provision a load balancer for the quiz frontend.

 ## Steps 

 1. Clone source code in Cloud Shell && Create Soft Link to the Repository
  - git clone https://github.com/GoogleCloudPlatform/training-data-analyst

  - ln -s ~/training-data-analyst/courses/developingapps/v1.2/nodejs/containerengine ~/containerengine

 2. Configure the case study application and review code 

  - cd ~/containerengine/start && . prepare_environment.sh

 3. Create a Kubernetes Engine Cluster with all GCP scope access 
   - gcloud container clusters create quiz-cluster --zone us-central1-b --scopes bigquery, cloud-platform, cloud-source-repos, cloud-source-repos-ro, compute-ro, compute-rw, datastore, default, gke-default, logging-write, monitoring, monitoring-read, monitoring-write, pubsub, service-control, service-management, sql-admin, storage-full, trace, userinfo-email

 4. Create the Dockerfile for the frontend and backend
   -- cd frontend/ && touch Dockerfile 

   Insert the following into file 
   
   FROM gcr.io/google_appengine/nodejs
   RUN /usr/local/bin/install_node '>=0.12.7'
   COPY . /app/
   RUN npm install -g npm@6.11.3 --unsafe-perm || \
   ((if [ -f npm-debug.log ]; then \
      cat npm-debug.log; \
    fi) && false)
   RUN npm update    
   CMD npm start
   
   Replicate file in Backend folder
   -- cp -r Dockerfile ../backend

   Move to the previous directory `../`

  5. Build Docker images with Cloud Build
   - gcloud builds submit -t gcr.io/qwiklabs-gcp-04-aeae1a137866/quiz-frontend ./frontend/

   - gcloud builds submit -t gcr.io/qwiklabs-gcp-04-aeae1a137866/quiz-backend ./backend/

  6. Create a Kubernetes Deployment file
      - In the frontend-deployment.yaml file, paste in the following:

         apiVersion: apps/v1beta1
         kind: Deployment
         metadata:
         name: quiz-frontend
         labels:
         app: quiz-app
         spec:
            replicas: 3
            template:
               metadata:
                  labels:
                     app: quiz-app
                     tier: frontend
         spec:
            containers:
            - name: quiz-frontend
               image: gcr.io/qwiklabs-gcp-04-aeae1a137866/quiz-frontend
               imagePullPolicy: Always
               ports:
               - name: http-server
                  containerPort: 8080
               env:
                  - name: GCLOUD_PROJECT
                  value: qwiklabs-gcp-04-aeae1a137866
                  - name: GCLOUD_BUCKET
                  value: qwiklabs-gcp-04-aeae1a137866-media
                  - name: NODE_ENV
                  value: production
      
      - In the backend-deployment.yaml file, paste in the following : 

         apiVersion: apps/v1beta1
            kind: Deployment
            metadata:
            name: quiz-backend
            labels:
            app: quiz-app
            spec:
               replicas: 3
               template:
                  metadata:
                     labels:
                        app: quiz-app
                        tier: backend
            spec:
               containers:
               - name: quiz-backend
                  image: gcr.io/qwiklabs-gcp-04-aeae1a137866/quiz-backend
                  imagePullPolicy: Always
                  ports:
                  - name: http-server
                     containerPort: 8080
                  env:
                     - name: GCLOUD_PROJECT
                     value: qwiklabs-gcp-04-aeae1a137866
                     - name: GCLOUD_BUCKET
                     value: qwiklabs-gcp-04-aeae1a137866-media
                     - name: NODE_ENV
                     value: production

  7. Execute the Deployment and Service Files
     - In Cloud Shell, provision the quiz frontend Deployment.
       - kubectl create -f ./frontend-deployment.yaml
      
     - In Cloud Shell, provision the quiz frontend Deployment
       - kubectl create -f ./frontend-service.yaml

   Test and see pods running:
   `Kubectl get pods` && `kubectl get all`


# LAB : App Dev - Deploying the Application into App Engine Flexible Environment: Node.js

# Objectives : 
   
In this lab, you learn how to perform the following tasks:

 - Create an app.yaml file to describe the App Engine Flex requirements for an application.
 
 - Deploy the quiz application into App Engine Flex.
 
 - Employ versions and traffic splitting to perform A/B testing of an application feature.

 ## Steps : 

 1. Clone the repository for the class.
   
    - git clone https://github.com/GoogleCloudPlatform/training-data-analyst
 
 2. Create a soft link as a shortcut to the working directory:
   -  ln -s ~/training-data-analyst/courses/developingapps/v1.2/nodejs/appengine ~/appengine

 3. Change the directory that contains the sample files for this lab.
   - cd ~/appengine/start

 4. Configure the Quiz application.
   - . prepare_environment.sh

 5. Create the app.yaml file for the frontend
   - runtime: nodejs \ env: flex   >> ./frontend/app.yaml

 6. Modify frontend config file 
   - { "GCLOUD_BUCKET" : "qwiklabs-gcp-03-3740d02596d7"} >> ./frontend/config.json

 7. Deploy the frontend to App Engine Flex
   - gcloud app deploy ./frontend/app.yaml

 8. Update the quiz application
     Open home.pug
         - vim frontend/web-app/views/home.pug

      Insert the following: 
         extends base.pug

         block content
         h1 Welcome to the Quite Interesting Quiz!!!!!!!!!!
         .jumbotron
            p Welcome to the Quite Interesting Quiz where you can create a question, take a test or review feedback
         h3.col-md-4
            a(href="/questions/add") Create Question
         h3.col-md-4
            a(href="/client/") Take Test
         h3.col-md-4
            a(href="/leaderboard") Leaderboa

 9. Deploy the updated application
   - gcloud app deploy ./frontend/app.yaml --no-promote \ --no-stop-previous-version

     