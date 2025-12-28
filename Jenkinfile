pipeline {
    agent { label 'sonu' }
    
    tools {
        maven 'mymaven'
    }
    
    stages {
        stage('Code Checkout') {
            steps {
                git url: 'https://github.com/RAVITEJA3929/python-code-library-app.git'
                script {
                    env.GIT_BRANCH = sh(script: 'git branch --show-current', returnStdout: true).trim()
                    env.GIT_COMMIT = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'mysonar'
                    withSonarQubeEnv('mysonar') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=python-library-app \
                            -Dsonar.projectName="Python Library App" \
                            -Dsonar.sources=. \
                            -Dsonar.python.version=3
                        """
                    }
                }
            }
        }
        
        stage('Build Docker Images') {
            steps {
                sh '''
                    docker build -t raviteja3929/dbimage:${BUILD_NUMBER} database/
                    docker build -t raviteja3929/authimage:${BUILD_NUMBER} auth/
                    docker build -t raviteja3929/bookimage:${BUILD_NUMBER} book/
                    docker build -t raviteja3929/borrowimage:${BUILD_NUMBER} borrow/
                    docker build -t raviteja3929/frontendimage:${BUILD_NUMBER} .
                '''
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([ 
                    credentialsId: 'docker-cred',
                    url: '' 
                ]) {
                    sh '''
                        docker tag raviteja3929/dbimage:${BUILD_NUMBER} raviteja3929/dbimage:latest
                        docker tag raviteja3929/authimage:${BUILD_NUMBER} raviteja3929/authimage:latest
                        docker tag raviteja3929/bookimage:${BUILD_NUMBER} raviteja3929/bookimage:latest
                        docker tag raviteja3929/borrowimage:${BUILD_NUMBER} raviteja3929/borrowimage:latest
                        docker tag raviteja3929/frontendimage:${BUILD_NUMBER} raviteja3929/frontendimage:latest
                        
                        docker push raviteja3929/dbimage:${BUILD_NUMBER}
                        docker push raviteja3929/dbimage:latest
                        docker push raviteja3929/authimage:${BUILD_NUMBER}
                        docker push raviteja3929/authimage:latest
                        docker push raviteja3929/bookimage:${BUILD_NUMBER}
                        docker push raviteja3929/bookimage:latest
                        docker push raviteja3929/borrowimage:${BUILD_NUMBER}
                        docker push raviteja3929/borrowimage:latest
                        docker push raviteja3929/frontendimage:${BUILD_NUMBER}
                        docker push raviteja3929/frontendimage:latest
                    '''
                }
            }
        }
        
        stage('Deploy Docker Swarm Stack') {
            steps {
                sh '''
                    docker stack deploy -c compose.yml teja_stack --with-registry-auth
                    sleep 30
                    docker stack ps teja_stack --no-trunc
                    docker service ls | grep teja_stack
                '''
            }
        }
    }
    
    post {
    always {
        mail to: 'kanchanapallyraviteja8@gmail.com',
            subject: "BUILD ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            body: """
*${currentBuild.currentResult}:* Job ${env.JOB_NAME}
Build #${env.BUILD_NUMBER}
Branch: ${env.GIT_BRANCH}
Duration: ${currentBuild.durationString}

More info at: ${env.BUILD_URL}console
            """
    }
}

}
