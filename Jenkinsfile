pipeline {
    agent any

    tools {
        go 'Go 1.18' // Make sure this matches Jenkins Global Tools name
    }

    environment {
        PROJECT_NAME = 'go-nexus-sample'
        NEXUS_URL = 'http://52.23.58.40:8081'
        NEXUS_REPO = 'go-test'
        NEXUS_CREDENTIALS_ID = 'NEXUS-CRED' // Jenkins credentials ID for Nexus username + password (123456)
    }

    stages {

        stage(' Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Venkatvicky/go-nexus-sample.git'
            }
        }

        stage(' Build Go App') {
            steps {
                sh 'go mod tidy'
                sh 'go build -o ${PROJECT_NAME}'
            }
        }

        stage(' Archive Go Artifact') {
            steps {
                sh '''
                    which zip || (sudo apt-get update && sudo apt-get install zip -y)
                    mkdir -p output
                    mv ${PROJECT_NAME} output/
                    cd output
                    zip ${PROJECT_NAME}.zip ${PROJECT_NAME}
                '''
            }
        }

        stage(' Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDENTIALS_ID}", passwordVariable: 'NEXUS_PASS', usernameVariable: 'NEXUS_USER')]) {
                    sh '''
                        curl -v -u $NEXUS_USER:$NEXUS_PASS \
                        --upload-file output/${PROJECT_NAME}.zip \
                        ${NEXUS_URL}/repository/${NEXUS_REPO}/${PROJECT_NAME}.zip
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo " Build or upload failed!"
        }
        success {
            echo " Go module successfully uploaded to Nexus!"
        }
    }
}
