pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build'
                archiveArtifacts artifacts: 'src/index.html'
            }

        stage('Scan image with Aqua') {
        aqua locationType: 'local', localImage: 'demouser/devops:${BUILD_NUMBER}', hideBase: false,  notCompliesCmd: '', onDisallowed: 'fail', showNegligible: false
        }

        stage('Push to registry') {
        // Going to skip this part for testing.
        }

        stage('Scan image with Aqua') {
        aqua locationType: 'local', localImage: 'demouser/devops:${BUILD_NUMBER}', hideBase: false,  notCompliesCmd: '', onDisallowed: 'fail', showNegligible: false, register: true, registry: "Docker Hub"
        }

        stage('DeployToStage') {
            when {
                branch 'master'
            }
            steps {
                withCredentials([string(credentialsId: 'cloud_user_pw', variable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: 'cloud_user',
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'src/**',
                                        removePrefix: 'src/'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
        stage('DeployToProd') {
            when {
                branch 'master'
            }
            steps {
                input 'Does the staging environment look OK?'
                milestone(1)
                withCredentials([string(credentialsId: 'cloud_user_pw', variable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'production',
                                sshCredentials: [
                                    username: 'cloud_user',
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'src/**',
                                        removePrefix: 'src/'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
 }
