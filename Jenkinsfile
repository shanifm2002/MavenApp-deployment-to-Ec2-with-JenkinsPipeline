pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        // This is the standard path for the Ubuntu openjdk-21-jdk package
        JAVA_HOME = '/usr/lib/jvm/java-21-openjdk-amd64'
        PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
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
                // This MUST return a version now that you've installed the JDK
                sh 'java -version'
                sh 'javac -version'
                sh 'mvn -version'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
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
                    '''
                }
            }
        }
    }
}
