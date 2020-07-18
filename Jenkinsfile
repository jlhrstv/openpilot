def remote = [:]
remote.user = "root"
remote.port = 8022
remote.retryCount = 5
remote.retryWaitSec = 5
remote.allowAnyHosts = true
remote.agent = true

pipeline {
  agent {
    docker {
      image 'python:3.7.3'
      args '--user=root'
    }
  }
  environment {
    COMMA_JWT = credentials('athena-test-jwt')
  }

  stages {

    stage('SSH test') {
      steps {
        lock(resource: "", label: 'eon-test', inversePrecedence: true, variable: 'eon_ip', quantity: 1){
          withCredentials([sshUserPrivateKey(credentialsId: 'id_rsa', keyFileVariable: 'id_file')]) {
            script {
              remote.name = eon_ip
              remote.host = eon_ip
              remote.identityFile = id_file
            }
            sshCommand remote: remote, command: "echo /VERSION"
          }
        }
      }
    }

    /*
    stage('Release Build') {
      when {
        branch 'devel-staging'
      }
      steps {
        lock(resource: "", label: 'eon-build', inversePrecedence: true, variable: 'eon_ip', quantity: 1){
          timeout(time: 60, unit: 'MINUTES') {
            dir(path: 'selfdrive/test') {
              sh 'pip install paramiko'
              sh 'python phone_ci.py "cd release && PUSH=1 ./build_release2.sh"'
            }
          }
        }
      }
    }

    stage('On-device Tests') {
      when {
        not {
          anyOf {
            branch 'master-ci'; branch 'devel'; branch 'devel-staging'; branch 'release2'; branch 'release2-staging'; branch 'dashcam'; branch 'dashcam-staging'
          }
        }
      }

      parallel {

        stage('Build') {
          environment {
            CI_PUSH = "${env.BRANCH_NAME == 'master' ? 'master-ci' : ''}"
          }

          steps {
            lock(resource: "", label: 'eon', inversePrecedence: true, variable: 'eon_ip', quantity: 1){
              timeout(time: 60, unit: 'MINUTES') {
                dir(path: 'selfdrive/test') {
                  sh 'pip install paramiko'
                  sh 'python phone_ci.py "cd release && ./build_devel.sh"'
                }
              }
            }
          }
        }

        stage('Replay Tests') {
          steps {
            lock(resource: "", label: 'eon2', inversePrecedence: true, variable: 'eon_ip', quantity: 1){
              timeout(time: 60, unit: 'MINUTES') {
                dir(path: 'selfdrive/test') {
                  sh 'pip install paramiko'
                  sh 'python phone_ci.py "cd selfdrive/test/process_replay && ./camera_replay.py"'
                }
              }
            }
          }
        }

        stage('HW Tests') {
          steps {
            lock(resource: "", label: 'eon', inversePrecedence: true, variable: 'eon_ip', quantity: 1){
              timeout(time: 60, unit: 'MINUTES') {
                dir(path: 'selfdrive/test') {
                  sh 'pip install paramiko'
                  sh 'python phone_ci.py "SCONS_CACHE=1 scons -j3 cereal/ && \
                                          nosetests -s selfdrive/test/test_sounds.py && \
                                          nosetests -s selfdrive/boardd/tests/test_boardd_loopback.py"'
                }
              }
            }
          }
        }

      }
    }
    */

  }
}
