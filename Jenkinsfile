pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner' // [CHANGE] SonarQube Scanner tool name in Jenkins
        DOCKER_IMAGE = "jaiaradhye/vulscanner:latest" // [CHANGE] DockerHub image name
        SONARQUBE_URL = "http://3.108.81.135:9000/" // [CHANGE] SonarQube server URL
        SONARQUBE_TOKEN = "squ_b8df9e36659dc7f2dc634000b4f7d7464bd9d534" // [CHANGE] SonarQube authentication token
        KUBE_CONFIG = "~/.kube/config" // [CHANGE] Path to Kubernetes config file
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Jaiaradhye/vulscannerProjectCI-CD.git' // [CHANGE] Your GitHub repo URL
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') { // [CHANGE] SonarQube server name from Jenkins settings
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=NessusScanner \  # [CHANGE] Unique project key in SonarQube
                        -Dsonar.sources=. \ # [DO NOT CHANGE] Scan all source files
                        -Dsonar.host.url=$SONARQUBE_URL \ # [CHANGE] SonarQube server URL
                        -Dsonar.login=$SONARQUBE_TOKEN # [CHANGE] SonarQube authentication token
                    '''
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                script {
                    def qualityGate = waitForQualityGate() // [DO NOT CHANGE] Checks SonarQube quality gate status
                    if (qualityGate.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub-credentials', url: '']) { // [CHANGE] DockerHub credentials in Jenkins
                    sh '''
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl --kubeconfig=$KUBE_CONFIG apply -f k8s-nessus-deployment.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl --kubeconfig=$KUBE_CONFIG get pods -l app=nessus
                kubectl --kubeconfig=$KUBE_CONFIG get services -l app=nessus
                '''
            }
        }

        stage('Monitor with Prometheus') {
            steps {
                sh '''
                kubectl --kubeconfig=$KUBE_CONFIG port-forward svc/prometheus 9090:9090 &
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for errors."
        }
    }
}
