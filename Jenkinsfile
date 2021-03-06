pipeline {
  agent {
    node {
      label "devops"
    }
  }
  stages {
    stage('Build') {
      steps {
        sh 'printenv'
      }
    }
    stage('Docker Build') {
      steps {
        script {
                sh 'docker build -t 10.1.3.114/pop-react:1.${BUILD_NUMBER} . '
        }
      }
    }

    stage ('Docker Publish') {
      steps {
        script {
                withDockerRegistry(credentialsId: 'dockerrepocred', url: 'http://10.1.3.114/') {
                    sh 'docker push 10.1.3.114/pop-react:1.${BUILD_NUMBER}'
                    sh 'docker rmi 10.1.3.114/pop-react:1.${BUILD_NUMBER}'
              }
            }
        }
      }
    

    stage ('K8S Deploy') {
      steps {
        script {
            sh '''#!/bin/sh
                    shopt -s nocasematch
                    BRANCH=$BRANCH_NAME

                    if [[ $BRANCH_NAME == master ]];
                    then
                        NAMESPACE="chitale"
                        CONTEXT="TEST"
                    elif [[ $BRANCH_NAME == release* ]];
                    then
                        NAMESPACE="chitale"
                        CONTEXT="PRD"
                    else
                        NAMESPACE=$BRANCH
                        CONTEXT="DEV"
                    fi
                    echo $NAMESPACE
                    echo $CONTEXT
                    NAMESPACE=${NAMESPACE//_/-}
                    shopt -u nocasematch
                    sed -i "s/NAMESPACE/$NAMESPACE/g" namespace.yaml
                    sed -i "s/NAMESPACE/$NAMESPACE/g" secret.yaml
                    sed -i "s/NAMESPACE/$NAMESPACE/g" chitalereactui.yaml
                    kubectl config use-context $CONTEXT
                    kubectl apply -f namespace.yaml
                    kubectl apply -f secret.yaml
                    kubectl apply -f chitalereactui.yaml 
                    kubectl set image deployment/pop-react pop-react=10.1.3.114/pop-react:1.${BUILD_NUMBER} -n dev --record=true'''
          
        }
      }
    }
  }
}
