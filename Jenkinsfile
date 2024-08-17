pipeline {
    agent any

    environment {
        ARTIFACTORY_URL = 'http://localhost:8081/artifactory'
        ARTIFACTORY_CRED = 'artifactory-credentials-id'
        MAVEN_HOME = tool name: 'Maven 3.9.6', type: 'maven'
        GITHUB_TOKEN = credentials('github-credentials-id')
    }

    stages {
        /*stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    withMaven(
                        maven: "${MAVEN_HOME}",
                        options: [artifactoryServer: [
                            url: "${ARTIFACTORY_URL}",
                            credentialsId: "${ARTIFACTORY_CRED}"
                        ]]
                    ) {
                        sh 'mvn clean package'
                    }
                }
            }
        }

        stage('Publish to Artifactory') {
            steps {
                script {
                    def server = Artifactory.server "${ARTIFACTORY_URL}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "libs-release-local/"
                            }
                        ]
                    }"""
                    def upload = server.upload spec: uploadSpec, credentialsId: "${ARTIFACTORY_CRED}"
                    upload.successfulBuild()
                }
            }
        }*/

        stage('Build') {
            steps {
                script {
                    sh '${MAVEN_HOME}/bin/mvn clean package'

                }
            }
        }
        stage('Deploy to GitHub Packages') {
                    steps {
                        script {
                            // Generate Maven settings.xml for GitHub Packages
                            writeFile file: 'settings.xml', text: '''
                            <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
                                        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                                        xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                                        http://maven.apache.org/SETTINGS/1.0.0/settings.xsd">
                                <servers>
                                    <server>
                                        <id>github</id>
                                        <username>vikramfa1</username>
                                        <password>github_token</password>
                                    </server>
                                </servers>
                            </settings>
                            '''
                            sh """
                            sed -i 's/github_token/${GITHUB_TOKEN}/' settings.xml
                            """
                            sh 'cat settings.xml'
                            // Deploy to GitHub Packages
                            sh '${MAVEN_HOME}/bin/mvn --settings settings.xml deploy'
                        }
                    }
                }
        stage('publish completion status to rule engine') {

                     steps {

                        sh """curl -X PUT -d '{"operationType":"JAVA_LIB","taskId":"JENKINS_PIPELINE","jobId":"2","jobName":"payment-util-lib","taskStatus":"COMPLETED"}' -H "Content-Type:application/json" http://host.docker.internal:8084/v1/job/2/update"""

                     }
                  }
    }



    post {
        success {
            echo 'Build and upload successful!'
        }
        failure {
            echo 'Build or upload failed.'
        }
    }
}
