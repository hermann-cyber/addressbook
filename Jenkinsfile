/*pipeline {
    agent { node { label "maven-sonarqube-node" } }   
    parameters {
      choice(name: 'aws_account',choices: ['999568710647', '4568366404742', '922266408974','576900672829'], description: 'aws account hosting image registry')
      choice(name: 'Environment', choices: ['Dev', 'QA', 'UAT', 'Prod'], description: 'Target environment for deployment')
      string(name: 'ecr_tag', defaultValue: '1.7.0', description: 'Assign the ECR tag version for the build')
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
                       -Dsonar.projectKey=addressbook-app \
                       -Dsonar.projectName='addressbook app' \
                      -Dsonar.host.url=http://35.89.134.204:9000 \
                       -Dsonar.token=${SONAR_TOKEN} \
                       -Dsonar.sources=src/main/java/ \
                       -Dsonar.java.binaries=target/classes \
                     """
                  }
              }
        }
    stage('4. Docker Image Build') {
      steps {
          sh "aws ecr-public get-login-password --region us-east-1  | sudo docker login --username AWS --password-stdin ${aws_account}.public.ecr.aws/f5z9p3h0"
          sh "docker build -t addressbook ."
          sh "sudo docker tag addressbook:latest ${aws_account}.public.ecr.aws/f5z9p3h0/addressbook ${params.ecr_tag}"
          sh "sudo docker push ${aws_account}.public.ecr.aws/f5z9p3h0/addressbook ${params.ecr_tag}"
      }
    }

    stage('5. Application Deployment in EKS') {
      steps {
        kubeconfig(caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '') {
          sh "kubectl apply -f manifest"
        }
      }
    }

    stage('6. Monitoring Solution Deployment in EKS') {
      steps {
        kubeconfig(caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '') {
          sh "kubectl apply -k monitoring"
          sh("""script/install_helm.sh""") 
          sh("""script/install_prometheus.sh""") 
        }
      }
    }

    stage('7. Email Notification') {
      steps {
        mail bcc: 'ngwahermann@gmail.com', body: '''Build is Over. Check the application using the URL below:
         https://app.dominionsystem.org/addressbook-1.0
         Let me/us know if the changes look okay.
         Thanks,
         Dominion System Technologies,
         +1 (313) 413-1477''', 
         subject: 'Application was Successfully Deployed!!', to: 'ngwahermann@gmail.com'
      }
    }
  }
}

*/
pipeline { 
    agent { node { label "maven-sonarqube-node" } }   
    parameters {
      choice(name: 'aws_account', choices: ['999568710647', '4568366404742', '922266408974', '576900672829'], description: 'AWS account hosting image registry')
      choice(name: 'Environment', choices: ['Dev', 'QA', 'UAT', 'Prod'], description: 'Target environment for deployment')
      string(name: 'ecr_tag', defaultValue: '1.7.0', description: 'Assign the ECR tag version for the build')
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
               -Dsonar.projectKey=addressbook-app \
               -Dsonar.projectName='addressbook app' \
               -Dsonar.host.url=http://35.89.134.204:9000 \
               -Dsonar.token=${SONAR_TOKEN} \
               -Dsonar.sources=src/main/java/ \
               -Dsonar.java.binaries=target/classes
            """
          }
        }
      }
      stage('4. Docker Image Build') {
        steps {
          sh "aws ecr-public get-login-password --region us-east-1  | docker login --username AWS --password-stdin ${aws_account}.711387106052.dkr.ecr.us-west-2.amazonaws.com"
          sh "docker build -t addressbook ."
          sh "docker tag addressbook:latest ${aws_account}.711387106052.dkr.ecr.us-west-2.amazonaws.com/addressbook:latest${params.ecr_tag}"
          sh "docker push ${aws_account}.711387106052.dkr.ecr.us-west-2.amazonaws.com/addressbook:latest${params.ecr_tag}"
        }
      }
      stage('5. Application Deployment in EKS') {
        steps {
          kubeconfig(caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '') {
            sh "kubectl apply -f manifest"
          }
        }
      }
      stage('6. Monitoring Solution Deployment in EKS') {
        steps {
          kubeconfig(caCertificate: '', credentialsId: 'kubeconfig', serverUrl: '') {
            sh "kubectl apply -k monitoring"
            sh("script/install_helm.sh") 
            sh("script/install_prometheus.sh")
          }
        }
      }
      stage('7. Email Notification') {
        steps {
          mail bcc: 'ngwahermann@gmail.com', body: '''Build is Over. Check the application using the URL below:
          https://app.dominionsystem.org/addressbook-1.0
          Let me/us know if the changes look okay.
          Thank,
          Dominion System Technologies,
          +1 (313) 413-1477''', 
          subject: 'Application was Successfully Deployed!!', to: 'ngwahermann@gmail.com'
        }
      }
    }
}
