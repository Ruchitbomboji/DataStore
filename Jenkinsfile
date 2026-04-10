pipeline {
    agent any

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
                sh """
                   echo "-------- Building Application --------"
                   mvn clean package
                   echo "------- Application Built Successfully --------"
                """
            }
        }

        stage("Maven Test") {
            steps {
                sh """
                    echo "-------- Executing Testcases --------"
                    mvn test
                    echo "-------- Testcases Execution Complete --------"
                """
            }
        }

        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh """
                        echo "-------- Running SonarQube Analysis --------"
                        mvn sonar:sonar \
                          -Dsonar.projectKey=DataStore \
                          -Dsonar.projectName=DataStore
                        echo "-------- SonarQube Analysis Complete --------"
                    """
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
                sh """
                  echo "-------- Pushing Artifacts To S3 --------"
                  aws s3 cp ./target/*.jar s3://datastore-ruchit-artefact-store-jenkins-apps/
                  echo "-------- Pushing Artifacts To S3 Completed --------"
                """
            }
        }
    }
}
