def phone(String ip, String cmd) {
  sh label: "phone: ${cmd}", script: "ssh -tt -o StrictHostKeyChecking=no -i selfdrive/test/id_rsa -p 8022 root@${ip} '${cmd}'"
}

def phone_script(String ip, String script) {
  sh label: "phone: ${script}", script: "ssh -tt -o StrictHostKeyChecking=no -i selfdrive/test/id_rsa -p 8022 -o SendEnv=GIT_COMMIT -o SendEnv=TEST_DIR root@${ip} < '${script}'"
}

def setup_environment(String ip) {
  phone_script(ip, "selfdrive/test/setup_phone_ci.sh")
}

pipeline {
  agent {
    docker {
      image 'python:3.7.3'
      args '--user=root'
    }
  }
  environment {
    COMMA_JWT = credentials('athena-test-jwt')
    TEST_DIR = "/data/openpilot/"
  }

  stages {

    stage('Release Build') {
      when {
        branch 'devel-staging'
      }
      steps {
        lock(resource: "", label: 'eon-build', inversePrecedence: true, variable: 'device_ip', quantity: 1){
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

        /*
        stage('Build') {
          environment {
            CI_PUSH = "${env.BRANCH_NAME == 'master' ? 'master-ci' : ''}"
          }

          steps {
            lock(resource: "", label: 'eon', inversePrecedence: true, variable: 'device_ip', quantity: 1){
              timeout(time: 60, unit: 'MINUTES') {
                dir(path: 'selfdrive/test') {
                  sh 'pip install paramiko'
                  sh 'python phone_ci.py "cd release && ./build_devel.sh"'
                }
              }
            }
          }
        }
        */

        stage('Replay Tests') {
          steps {
            lock(resource: "", label: 'eon2', inversePrecedence: true, variable: 'device_ip', quantity: 1){
              timeout(time: 60, unit: 'MINUTES') {
                setup_environment(device_ip)
                phone(device_ip, "printenv && cd $TEST_DIR/selfdrive/test/process_replay && CI=1 ./camera_replay.py")
              }
            }
          }
        }

        /*
        stage('HW Tests') {
          steps {
            lock(resource: "", label: 'eon', inversePrecedence: true, variable: 'device_ip', quantity: 1){
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
        */

      }
    }

  }
}
