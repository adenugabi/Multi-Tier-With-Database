pipeline {
    agent any
    
    tools {
        
        maven "maven"
    }
    environment {
        SONAR_HOME = "scanner"
    }
    stages {
        stage('1. Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/adenugabi/Multi-Tier-With-Database.git'
            }
        }
        stage('2. Compile source code') {
            steps {
                sh "mvn compile"
            }
        }
        stage('3. Test cases') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        stage('4. SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=Bankingapp -DskipTests=true"
                }
            }
        }

        stage('5. Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('6. Publish Artifacts To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-nexus-settings', jdk: '', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        stage('7. Docker Build') {
            steps {
                script{
                withDockerRegistry(credentialsId: 'docker-credential') {
                    sh "docker build -t bhisawlah/bankapp:latest ."
                }
                }
            }
        }
        stage('8. Docker Push') {
            steps {
                withDockerRegistry(credentialsId: 'docker-credential', url: 'https://index.docker.io/v1/') {
                    sh "docker push bhisawlah/bankapp:latest"
                }
            }
        }
        stage('9. Deploy to Kubernetes') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'project-cluster', contextName: '', credentialsId: 'k8s-credential', namespace: 'webapps', serverUrl: 'https://E73F31947340FAC745FFBB2BD5444725.gr7.eu-west-2.eks.amazonaws.com']]) {
                    sh "kubectl apply -f deployment.yaml"
                    sleep(30)
                }
            }
        }
        stage('10. Verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'project-cluster', contextName: '', credentialsId: 'k8s-credential', namespace: 'webapps', serverUrl: 'https://E73F31947340FAC745FFBB2BD5444725.gr7.eu-west-2.eks.amazonaws.com']]) {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
        stage('11. Email notification') {
    steps {
        script {
            def kubectlOutput = sh(script: 'kubectl get svc -n webapps', returnStdout: true).trim()


            emailext(
                body: """Welldone!!! your deployment was successful

                Here is the current status of services in the webapps namespace:

                
                ${kubectlOutput}
                        """,
                        subject: 'Deployment Successful',
                        to: 'ajisegbedeabisolat@gmail.com'
                    )
                }
            }
        }
    }
}