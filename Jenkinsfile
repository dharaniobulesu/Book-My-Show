pipeline {
  agent any
  tools {
    nodejs 'nodejs'
}

environment {
    SCANNER_HOME = tool 'sonar-scanner'
    IMAGE_NAME   = "dharaniobulesu/bookmyshow:latest"
}

stages {

    stage('Clean Workspace') {
        steps {
            cleanWs()
        }
    }

    stage('Checkout from Git') {
        steps {
            git branch: 'main', url: 'https://github.com/dharaniobulesu/Book-My-Show.git'
            sh 'ls -la'
        }
    }

    stage('SonarQube Analysis') {
        steps {
            withSonarQubeEnv('sonar-server') {
                sh '''
                $SCANNER_HOME/bin/sonar-scanner \
                -Dsonar.projectName=Book-My-Show \
                -Dsonar.projectKey=Book-My-Show
                '''
            }
        }
    }

    stage('Quality Gate') {
        steps {
            script {
                waitForQualityGate abortPipeline: false
            }
        }
    }

    stage('Install Dependencies') {
        steps {
            sh '''
            cd bookmyshow-app

            echo "Checking project files..."
            ls -la

            if [ -f package.json ]; then
                echo "Installing dependencies..."
                rm -rf node_modules package-lock.json
                npm install
            else
                echo "ERROR: package.json not found!"
                exit 1
            fi
            '''
        }
    }

    stage('Docker Build & Push') {
        steps {
            script {
                withDockerRegistry(credentialsId: 'dockerhub') {
                    sh '''
                    echo "Building Docker Image..."
                    docker build -t $IMAGE_NAME -f bookmyshow-app/Dockerfile bookmyshow-app

                    echo "Pushing Image to DockerHub..."
                    docker push $IMAGE_NAME
                    '''
                }
            }
        }
    }

    stage('Deploy Container') {
        steps {
            sh '''
            echo "Stopping old container..."
            docker stop bms || true
            docker rm bms || true

            echo "Starting new container..."
            docker run -d \
                --restart=always \
                --name bms \
                -p 3000:3000 \
                $IMAGE_NAME

            echo "Running containers:"
            docker ps -a
            '''
        }
    }
}

post {
    always {
        emailext attachLog: true,
            subject: "Build Result: ${currentBuild.result}",
            body: """


Project: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Build URL: ${env.BUILD_URL}
""",
to: '[dharanin022@gmail.com](mailto:dharanin022@gmail.com)'
}
}
}

