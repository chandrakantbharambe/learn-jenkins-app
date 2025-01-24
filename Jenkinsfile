pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    npm --version 
                    node --version
                    npm ci
                    npm run build
                    ls -la
                    npm test
                '''

                script {
                    def filePath = '/path/to/your/folder/yourfile.txt' // specify the path to your file
                    if (new File(filePath).exists()) {
                        echo "File exists: ${filePath}"
                    } else {
                        echo "File does not exist: ${filePath}"
                        // Optionally, you can fail the build
                        error("Required file not found!")
                    }
                }
            }
        }
    }
}