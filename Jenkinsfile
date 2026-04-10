pipeline {
    agent any

    parameters {
        string(name: "App_Version", description: "Provide application version")
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials("dockerhub-creds")
    }

    stages {

        stage("Checkout") {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/Ruchitbomboji/DataStore.git']]
                )
            }
        }

        stage("Maven Build") {
            steps {
                sh '''
                    echo "-------- Building Application --------"
                    mvn clean package
                    echo "------- Application Built Successfully --------"
                '''
            }
        }

        stage("Maven Test") {
            steps {
                sh '''
                    echo "-------- Executing Testcases --------"
                    mvn test
                    echo "-------- Testcases Execution Complete --------"
                '''
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''
                        echo "-------- Running SonarQube Analysis --------"
                        mvn sonar:sonar \
                          -Dsonar.projectKey=DataStore \
                          -Dsonar.projectName=DataStore
                        echo "-------- SonarQube Analysis Complete --------"
                    '''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Artifact Store") {
            steps {
                sh '''
                    echo "-------- Pushing Artifacts To S3 --------"
                    aws s3 cp ./target/*.jar s3://datastore-ruchit-artefact-store-jenkins-apps/
                    echo "-------- Pushing Artifacts To S3 Completed --------"
                '''
            }
        }

        stage("Docker Image Build") {
            steps {
                sh """
                    echo "-------- Building Docker Image --------"
                    docker build -t datastore:${params.App_Version} .
                    echo "-------- Image Successfully Built --------"
                """
            }
        }

        stage("Docker Image Scan") {
            steps {
                sh """
                    echo "-------- Scanning Docker Image --------"
                    trivy image datastore:${params.App_Version}
                    echo "-------- Scanning Docker Image Complete --------"
                """
            }
        }

        stage("Docker Image Tag") {
            steps {
                sh """
                    echo "-------- Tagging Docker Image --------"
                    docker tag datastore:${params.App_Version} ruchit27/jenkins-datastore:${params.App_Version}
                    echo "-------- Tagging Docker Image Completed --------"
                """
            }
        }

        stage("Loggingin & Pushing Docker Image") {
            steps {
                sh """
                    echo "-------- Logging To DockerHub --------"
                    echo "\$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "\$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                    echo "-------- DockerHub Login Successful --------"

                    echo "-------- Pushing Docker Image To DockerHub --------"
                    docker push ruchit27/jenkins-datastore:${params.App_Version}
                    echo "-------- Docker Image Pushed Successfully --------"
                """
            }
        }

        stage("Cleanup") {
            steps {
                sh """
                    echo "-------- Cleaning Up Jenkins Machine --------"
                    docker image prune -a -f
                    echo "-------- Clean Up Successful --------"
                """
            }
        }
    }
}
