pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo '*******************************************************************'
		echo '**                Running build automation                       **'
		echo '*******************************************************************'
		sh './gradlew build --no-daemon'
            }
        }
	stage('Archive Artifact') {
            steps {
		echo '*******************************************************************'
		echo '**                        Archive Artifact                       **'
		echo '*******************************************************************'
		archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
		    
		echo '*******************************************************************'
		echo '**                      Build Docker Image                       **'
		echo '*******************************************************************'
                script {
                    app = docker.build("pradeepe/train-schedule")
                    app.inside {
			echo '*******************************************************************'
			echo '**                         smoke test                            **'
			echo '*******************************************************************'
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
		    echo '*******************************************************************'
		    echo '**                    Push Docker Image                          **'
		    echo '*******************************************************************'
                    docker.withRegistry('https://registry.hub.docker.com', 'DockerHub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
	stage('DeployToStaging') {
	    when {
                branch 'master'
            }
            steps {
		echo '*******************************************************************'
		echo '**                    Deploy To Staging                          **'
		echo '*******************************************************************'
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'staging',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'dist/trainSchedule.zip',
                                        removePrefix: 'dist/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule'
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
