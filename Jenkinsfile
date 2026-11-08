
pipeline {
    agent any

    environment {
        MAVEN_HOME = '/opt/maven'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/iamsaketranjan/hello-world.git'
            }
        }

        stage('Check Tools') {
            steps {
                sh '''
                    echo "Checking Java version..."
                    java -version

                    echo "Checking Maven version..."
                    "$MAVEN_HOME/bin/mvn" -version
                '''
            }
        }

        stage('Build WAR') {
            steps {
                sh '''
                    echo "Building application using Maven..."
                    "$MAVEN_HOME/bin/mvn" clean package
                '''
            }
        }

        stage('Verify WAR') {
            steps {
                sh '''
                    echo "Checking generated WAR file..."

                    WAR_FILE=$(find webapp/target \
                        -maxdepth 1 \
                        -type f \
                        -name '*.war' \
                        -print -quit)

                    if [ -z "$WAR_FILE" ]; then
                        echo "ERROR: WAR file not found!"
                        exit 1
                    fi

                    echo "WAR file found:"
                    ls -lh "$WAR_FILE"

                    echo "WAR verification successful."
                '''
            }
        }

        
stage('Transfer and Deploy') {
    steps {
        sshPublisher(
            publishers: [
                sshPublisherDesc(
                    configName: 'dockerhost',
                    transfers: [
                        // Transfer the generated WAR.
                        sshTransfer(
                            sourceFiles: 'webapp/target/*.war',
                            removePrefix: 'webapp/target',
                            remoteDirectory: 'opt/docker'
                        ),

                        // Transfer Docker configuration and deploy.
                        sshTransfer(
                            sourceFiles: 'docker/Dockerfile,docker/manager-context.xml',
                            removePrefix: 'docker',
                            remoteDirectory: 'opt/docker',
                            execCommand: '''
                                set -e

                                echo "Checking transferred files..."
                                ls -lh /home/dockermain/opt/docker/

                                echo "Copying deployment files..."
                                sudo /usr/bin/cp \
                                    /home/dockermain/opt/docker/*.war \
                                    /opt/docker/

                                sudo /usr/bin/cp \
                                    /home/dockermain/opt/docker/Dockerfile \
                                    /opt/docker/Dockerfile

                                sudo /usr/bin/cp \
                                    /home/dockermain/opt/docker/manager-context.xml \
                                    /opt/docker/manager-context.xml

                                cd /opt/docker

                                echo "Checking Tomcat credentials configuration..."
                                test -f tomcat-users.xml

                                echo "Building Docker image..."
                                sudo /usr/bin/docker build \
                                    --no-cache \
                                    -f Dockerfile \
                                    -t regapp:v2 .

                                echo "Removing previous container..."
                                sudo /usr/bin/docker rm -f registryapp || true

                                echo "Starting new Tomcat container..."
                                sudo /usr/bin/docker run -d \
                                    --name registryapp \
                                    -p 8088:8080 \
                                    regapp:v2

                                echo "Verifying application..."

                                for attempt in $(seq 1 10); do
                                    if curl -fsS \
                                        -o /dev/null \
                                        http://localhost:8088/webapp/; then

                                        echo "Application health check passed: HTTP 200"
                                        exit 0
                                    fi

                                    echo "Waiting for application... attempt $attempt"
                                    sleep 3
                                done

                                echo "Application health check failed."
                                exit 1
                            '''
                        )
                    ],
                    verbose: true
                )
            ]
        )
    }
}


    post {
        success {
            echo 'SUCCESS: Application built, transferred, deployed and verified!'
        }

        failure {
            echo 'FAILURE: Pipeline execution failed. Check the failed stage in Console Output.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}
