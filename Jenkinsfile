pipeline {

    agent { node { label "maven-sonarqube-node" } }   

    parameters {

      choice(name: 'Environment', choices: ['Dev', 'QA', 'UAT', 'Prod'], description: 'Target environment for deployment')

      string(name: 'ecr_tag', defaultValue: '1.0.0', description: 'Assign the ECR tag version for the build')

    }
 
    tools {

      maven "Maven-3.9.8"

    }
 
    stages {

    stage('1. Git Checkout') {

      steps {

       git branch: 'release', credentialsId: 'PAT-project', url: 'https://github.com/hermann-cyber/addressbook.git'

      }

    }

    stage('2. Build with Maven') { 

      steps {

        sh "mvn clean package"

      }

    }

    stage('3. SonarQube Analysis') {

          environment {

                scannerHome = tool 'SonarQube-Scanner-6.2.1'

            }

            steps {

              withCredentials([string(credentialsId: 'sonarqube-credentials', variable: 'SONAR_TOKEN')]) {

                      sh """

                      ${scannerHome}/bin/sonar-scanner  \

                      -Dsonar.projectKey=addressbook \

                      -Dsonar.projectName='addressbook' \

                      -Dsonar.host.url=http://35.167.215.27:9000 \

                      -Dsonar.token=${SONAR_TOKEN} \

                      -Dsonar.sources=src/main/java/ \

                      -Dsonar.java.binaries=target/classes \

                     """

                  }

              }

        }

    stage('4. Docker Image Build') {

      steps {

        sh "aws ecr-public get-login-password --region us-east-1 | sudo docker login --username AWS --password-stdin public.ecr.aws/f5z9p3h0"

        sh "sudo docker build -t addressbook ."

        sh "sudo docker tag addressbook:latest public.ecr.aws/f5z9p3h0/addressbook:${params.ecr_tag}"

        sh "sudo docker push public.ecr.aws/f5z9p3h0/addressbook:${params.ecr_tag}"

      }

    }
 
    stage('5. Application Deployment in EKS') {

      steps {

        withKubeConfig([caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '']) {

          sh "kubectl apply -f manifest"

        }

      }

    }
 
    stage('6. Monitoring Solution Deployment in EKS') {

      steps {

        withKubeConfig([caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '']) {

          sh "kubectl apply -k monitoring"

          sh("""script/install_helm.sh""") 

          sh("""script/install_prometheus.sh""") 

        }

      }

    }
 
    stage('7. Email Notification') {

      steps {

        mail bcc: 'ngwahermann@gmail.com', body: '''Build is Over. Check the application using the URL below:
http://addressbook-server-ip-1575406555.us-west-2.elb.amazonaws.com/addressbook-1.0/

         Let me know if the changes look okay.

         Thanks,

         Team-B

         Dominion System Technologies,

         +44 7********6''', 

         subject: 'Application was Successfully Deployed!!', to: ''

      }

    }

  }

}
 
