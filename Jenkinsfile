pipeline{
    agent any
    tools {
        maven "Maven3.6.3"
    }
    environment{
      PASS = credentials('docker-hub')
      USERHUB = "${env.USER_HUB}"
      COMMIT = "${env.BUILD_NUMBER}"
    }
    stages {
        stage("git checkout") {
            steps {
              git "https://github.com/yankils/hello-world"
            }
        }

        stage("Build"){
            steps{
                sh "mvn -version"
                sh "mvn clean install"
            }
        }
        stage("ansible container"){
            steps([$class: 'BapSshPromotionPublisherPlugin']) {
                sshPublisher(
                  continueOnError: false, failOnError: true,
                  publishers: [
                    sshPublisherDesc(
                      configName: "ansible-server",
                      verbose: true,
                      transfers: [
                        sshTransfer(
                          sourceFiles: "webapp/target/*.war",
                          remoteDirectory: "//opt//docker",
                          removePrefix: "webapp/target",
                          execCommand: "ansible-playbook //opt/docker/create-simple-devops-image.yml --limit localhost --extra-vars '{\"userFromJenkins\":$USERHUB,\"pass\":$PASS,\"imageTag\":$COMMIT}'"
                        ),
                        sshTransfer(
                          execCommand: "ansible-playbook //opt/docker/create-simple-devops-project.yml --limit 192.168.0.201 --extra-vars '{\"userFromJenkins\":$USERHUB,\"pass\":$PASS,\"imageTag\":$COMMIT}'"
                        )
                      ]
                    )
                  ]
                )
            }
        }
    }
}
