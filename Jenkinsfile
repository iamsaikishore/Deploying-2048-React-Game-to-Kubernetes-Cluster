pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/iamsaikishore/Deploying-2048-React-Game-to-Kubernetes-Cluster.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                    -Dsonar.projectKey=Game '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
	stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
	stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t 2048 ."
		       sh "docker tag 2048 iamsaikishore/2048-game:v1.${env.BUILD_NUMBER}"
                       sh "docker tag 2048 iamsaikishore/2048-game:latest "
		       sh "docker push iamsaikishore/2048-game:v1.${env.BUILD_NUMBER}"
                       sh "docker push iamsaikishore/2048-game:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image iamsaikishore/2048-game:latest > trivy.txt" 
            }
        }
	stage('Deploy to container'){
            steps{
                sh 'docker run -d --name 2048 -p 3000:3000 iamsaikishore/2048-game:latest'
            }
        }

    }
}

