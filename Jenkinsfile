def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
    'UNSTABLE': 'danger'
]
pipeline {
  agent any
  environment {
    WORKSPACE = "${env.WORKSPACE}/realworld-cicd-pipeline-project-main"
    NEXUS_CREDENTIAL_ID = 'Nexus-Credential'
    //NEXUS_USER = "$NEXUS_CREDS_USR"
    //NEXUS_PASSWORD = "$Nexus-Token"
    //NEXUS_URL = "172.31.18.62:8081"
    //NEXUS_REPOSITORY = "maven_project"
    //NEXUS_REPO_ID    = "maven_project"
    //ARTVERSION = "${env.BUILD_ID}"
  }
  tools {
    maven 'localMaven'
    jdk 'localJdk'
  }
  stages {
    stage('Build') {
      steps {
        // dir('realworld-cicd-pipeline-project-main/') {
        sh 'mvn clean package'
      }
      post {
        success {
          echo ' now Archiving '
          archiveArtifacts artifacts: '**/*.war'
        }
      }
    }
    stage('Unit Test'){
        steps {
            // dir('realworld-cicd-pipeline-project-main/') {
            sh 'mvn test'
        }
    }
    stage('Integration Test'){
        steps {
            // dir('realworld-cicd-pipeline-project-main/') {
          sh 'mvn verify -DskipUnitTests'
        }
    }
    stage ('Checkstyle Code Analysis'){
        steps {
            // dir('realworld-cicd-pipeline-project-main/') {
            sh 'mvn checkstyle:checkstyle'
        }
        post {
            success {
                echo 'Generated Analysis Result'
            }
        }
    }
    stage('SonarQube Inspection') {
        steps {
            // dir('realworld-cicd-pipeline-project-main/') {
            withSonarQubeEnv('SonarQube') { 
                withCredentials([string(credentialsId: 'SonarQube-Token3', variable: 'SONAR_TOKEN')]) {
                sh """
                mvn sonar:sonar \
                -Dsonar.projectKey=JavaWebApp-Project5 \
                -Dsonar.host.url=http://172.31.0.111:9000 \
                -Dsonar.login=$SONAR_TOKEN
                """
                }
            }
        }
    }
    // stage('SonarQube GateKeeper') {
    //     steps {
    //       timeout(time : 1, unit : 'HOURS'){
    //       waitForQualityGate abortPipeline: true
    //       }
    //    }
    // }
    stage("Nexus Artifact Uploader"){
        steps{
            // dir('realworld-cicd-pipeline-project-main/') {
           nexusArtifactUploader(
              nexusVersion: 'nexus3',
              protocol: 'http',
              nexusUrl: '172.31.12.229:8081',
              groupId: 'webapp',
              version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
              repository: 'maven-project-releases5',  //"${NEXUS_REPOSITORY}",
              credentialsId: "${NEXUS_CREDENTIAL_ID}",
              artifacts: [
                  [artifactId: 'webapp',
                  classifier: '',
                  file: "${WORKSPACE}/webapp/target/webapp.war",
                  type: 'war']
              ]
           )
        }
    }
    stage('Deploy to Development Env') {
        environment {
            HOSTS = 'dev'
        }
        steps {
            // dir('realworld-cicd-pipeline-project-main/') {
            withCredentials([usernamePassword(credentialsId: 'Ansible-Credential', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                sh "ansible-playbook -i ${WORKSPACE}/ansible-config/aws_ec2.yaml ${WORKSPACE}/deploy.yaml --extra-vars \"ansible_user=$USER_NAME ansible_password=$PASSWORD hosts=tag_Environment_$HOSTS workspace_path=$WORKSPACE\""
            }
        }
    }
    stage('Deploy to Staging Env') {
        environment {
            HOSTS = 'stage'
        }
        steps {
            // dir('realworld-cicd-pipeline-project-main/') {
            withCredentials([usernamePassword(credentialsId: 'Ansible-Credential', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                sh "ansible-playbook -i ${WORKSPACE}/ansible-config/aws_ec2.yaml ${WORKSPACE}/deploy.yaml --extra-vars \"ansible_user=$USER_NAME ansible_password=$PASSWORD hosts=tag_Environment_$HOSTS workspace_path=$WORKSPACE\""
            }
        }
    }
    stage('Quality Assurance Approval') {
        steps {
            input('Do you want to proceed?')
        }
    }
    stage('Deploy to Production Env') {
        environment {
            HOSTS = 'prod'
        }
        steps {
            // dir('realworld-cicd-pipeline-project-main/') {
            withCredentials([usernamePassword(credentialsId: 'Ansible-Credential', passwordVariable: 'PASSWORD', usernameVariable: 'USER_NAME')]) {
                sh "ansible-playbook -i ${WORKSPACE}/ansible-config/aws_ec2.yaml ${WORKSPACE}/deploy.yaml --extra-vars \"ansible_user=$USER_NAME ansible_password=$PASSWORD hosts=tag_Environment_$HOSTS workspace_path=$WORKSPACE\""
            }
         }
      }
   }
  post {
    always {
        echo 'Slack Notifications.'
        slackSend channel: 'innovative-batch-pipeline-alert1', //update and provide your channel name
        color: COLOR_MAP[currentBuild.currentResult],
        message: "*${currentBuild.currentResult}:* Job Name '${env.JOB_NAME}' build ${env.BUILD_NUMBER} \n Build Timestamp: ${env.BUILD_TIMESTAMP} \n Project Workspace: ${env.WORKSPACE} \n More info at: ${env.BUILD_URL}"
    }
  }
}

