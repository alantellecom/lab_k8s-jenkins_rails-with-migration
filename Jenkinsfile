  
pipeline {
  
  environment {

    PROJECT = "rails-lab-264600"
    APP_NAME = "sample-app"
    CLUSTER = "standard-cluster-1"
    CLUSTER_ZONE = "us-central1-a"
    JENKINS_CRED = "${PROJECT}"
    TAG_IMAGE = "gcr.io/${PROJECT}/${APP_NAME}:${env.GIT_COMMIT}"
    JOBS_PATH = "k8s/jobs/"
    DEPLOY_PATH = "k8s/deploy/"
  }

  agent {
    kubernetes {
      label 'pipeline'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
}
  }

  stages {
    
    stage("Checkout code") {
      steps {
        checkout scm
      }
    }
    
        stage('Build and push image') {
      steps {
        container('gcloud') {
          sh "PYTHONUNBUFFERED=1 gcloud builds submit --config cloudbuild.yaml  --substitutions=TAG_NAME=${TAG_IMAGE} ."
        } 
      }
    }
    
    stage('Migration') {
      steps{
        container('kubectl') {  
          sh "sed -i 's#gcr.io/project/image-name#${TAG_IMAGE}#g'   ./${JOBS_PATH}* "   
          
          step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: env.JOBS_PATH, credentialsId: env.JENKINS_CRED, verifyDeployments: true])          
        }
      }
    }



    stage('Deploy') {
      steps{
        container('kubectl') {
          sh "sed -i 's#gcr.io/project/image-name#${TAG_IMAGE}#g' ./${DEPLOY_PATH}* "     
          step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT, clusterName: env.CLUSTER, zone: env.CLUSTER_ZONE, manifestPattern: env.DEPLOY_PATH, credentialsId: env.JENKINS_CRED, verifyDeployments: true])          
        }
      }
    }

  }

}
