pipeline {
    options {
      disableConcurrentBuilds()
    }  
    agent {
      label "jenkins-maven"
    }
    environment {
      ORG               = 'activiti'
      APP_NAME          = 'activiti-cloud-full-example'
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
      GITHUB_CHARTS_REPO    = "https://github.com/Activiti/activiti-cloud-helm-charts.git"
      GITHUB_HELM_REPO_URL = "https://activiti.github.io/activiti-cloud-helm-charts/"
      HELM_RELEASE_NAME = "modexample-$BRANCH_NAME-$BUILD_NUMBER".toLowerCase()

      PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
      PREVIEW_NAMESPACE = "moexample-$BRANCH_NAME-$BUILD_NUMBER".toLowerCase()
      
      REALM = "activiti"

    }
    stages {
      stage('CI Build and push snapshot') {
        when {
          branch 'PR-*'
        }
        environment {
         GATEWAY_HOST = "activiti-cloud-gateway.$PREVIEW_NAMESPACE.35.228.195.195.nip.io"
         SSO_HOST = "activiti-keycloak.$PREVIEW_NAMESPACE.35.228.195.195.nip.io"
        }
        steps {
          container('maven') {
           dir ("./charts/$APP_NAME") {
	           // sh 'make build'
              sh 'make install'
            }

            dir("./activiti-cloud-acceptance-scenarios") {
              git 'https://github.com/Activiti/activiti-cloud-acceptance-scenarios.git'
              sh 'sleep 120'
              sh "mvn clean install -DskipTests && mvn -pl 'modeling-acceptance-tests' clean verify"
            }
          }
        }
      }
      stage('Build Release') {
        when {
          branch 'master'
        }
	environment {
         GATEWAY_HOST = "activiti-cloud-gateway.$PREVIEW_NAMESPACE.35.228.195.195.nip.io"
         SSO_HOST = "activiti-keycloak.$PREVIEW_NAMESPACE.35.228.195.195.nip.io"
        }      
        steps {
          container('maven') {
            // ensure we're not on a detached head
            sh "git checkout master"
            sh "git config --global credential.helper store"

            sh "jx step git credentials"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"
            dir ("./charts/$APP_NAME") {
              sh 'make install'
            }
	   //run tests	
            dir("./activiti-cloud-acceptance-scenarios") {
              git 'https://github.com/Activiti/activiti-cloud-acceptance-scenarios.git'
              sh 'sleep 120'
              sh "mvn clean install -DskipTests && mvn -pl '!apps-acceptance-tests,!multiple-runtime-acceptance-tests' clean verify"
            }	  
	    //end run tests	  
          }
        }
      }
    }
   post {
        always {
          container('maven') {
            dir("./charts/$APP_NAME") {
               sh "make delete" 
            }
            sh "kubectl delete namespace $PREVIEW_NAMESPACE" 
          }
          cleanWs()
        }
   }
}
