pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'jdk-21' 
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/shanifm2002/MavenApp-deployment-to-Ec2-with-JenkinsPipeline.git'
            }
        }

        stage('Verify Environment') {
            steps {
                script {
                    def jdkHome = tool 'jdk-21'
                    withEnv(["JAVA_HOME=${jdkHome}", "PATH=${jdkHome}/bin:${env.PATH}"]) {
                        sh 'java -version'
                        sh 'javac -version'
                        sh 'mvn -version'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    def jdkHome = tool 'jdk-21'
                    // We force the PATH here so 'javac' is definitely version 21
                    withEnv(["JAVA_HOME=${jdkHome}", "PATH=${jdkHome}/bin:${env.PATH}"]) {
                        sh 'mvn clean package -DskipTests'
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                        echo "Copying artifact to EC2..."
                        scp -o StrictHostKeyChecking=no \
                            target/demo-1.0.0.jar \
                            ubuntu@18.232.187.168:/opt/app/

                        echo "Starting application on EC2..."
                        ssh -o StrictHostKeyChecking=no ubuntu@18.232.187.168 << 'EOF'
                            pkill -f demo-1.0.0.jar || true
                            nohup java -jar /opt/app/demo-1.0.0.jar > /opt/app/app.log 2>&1 &
EOF
                        echo "Deployment command executed successfully"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Successfully deployed to EC2."
        }
        failure {
            echo "Build failed. Ensure 'jdk-21' is correctly configured in Jenkins Global Tool Configuration."
        }
    }
}
