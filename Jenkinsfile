def remote = [:]
remote.user = "root"
remote.port = 8022
remote.retryCount = 5
remote.retryWaitSec = 5
remote.allowAnyHosts = true
remote.agent = true
remote.identity = """-----BEGIN RSA PRIVATE KEY-----
MIIEvAIBADANBgkqhkiG9w0BAQEFAASCBKYwggSiAgEAAoIBAQC+iXXq30Tq+J5N
Kat3KWHCzcmwZ55nGh6WggAqECa5CasBlM9VeROpVu3beA+5h0MibRgbD4DMtVXB
t6gEvZ8nd04E7eLA9LTZyFDZ7SkSOVj4oXOQsT0GnJmKrASW5KslTWqVzTfo2XCt
Z+004ikLxmyFeBO8NOcErW1pa8gFdQDToH9FrA7kgysic/XVESTOoe7XlzRoe/eZ
acEQ+jtnmFd21A4aEADkk00Ahjr0uKaJiLUAPatxs2icIXWpgYtfqqtaKF23wSt6
1OTu6cAwXbOWr3m+IUSRUO0IRzEIQS3z1jfd1svgzSgSSwZ1Lhj4AoKxIEAIc8qJ
rO4uymCJAgMBAAECggEBAISFevxHGdoL3Z5xkw6oO5SQKO2GxEeVhRzNgmu/HA+q
x8OryqD6O1CWY4037kft6iWxlwiLOdwna2P25ueVM3LxqdQH2KS4DmlCx+kq6FwC
gv063fQPMhC9LpWimvaQSPEC7VUPjQlo4tPY6sTTYBUOh0A1ihRm/x7juKuQCWix
Cq8C/DVnB1X4mGj+W3nJc5TwVJtgJbbiBrq6PWrhvB/3qmkxHRL7dU2SBb2iNRF1
LLY30dJx/cD73UDKNHrlrsjk3UJc29Mp4/MladKvUkRqNwlYxSuAtJV0nZ3+iFkL
s3adSTHdJpClQer45R51rFDlVsDz2ZBpb/hRNRoGDuECgYEA6A1EixLq7QYOh3cb
Xhyh3W4kpVvA/FPfKH1OMy3ONOD/Y9Oa+M/wthW1wSoRL2n+uuIW5OAhTIvIEivj
6bAZsTT3twrvOrvYu9rx9aln4p8BhyvdjeW4kS7T8FP5ol6LoOt2sTP3T1LOuJPO
uQvOjlKPKIMh3c3RFNWTnGzMPa0CgYEA0jNiPLxP3A2nrX0keKDI+VHuvOY88gdh
0W5BuLMLovOIDk9aQFIbBbMuW1OTjHKv9NK+Lrw+YbCFqOGf1dU/UN5gSyE8lX/Q
FsUGUqUZx574nJZnOIcy3ONOnQLcvHAQToLFAGUd7PWgP3CtHkt9hEv2koUwL4vo
ikTP1u9Gkc0CgYEA2apoWxPZrY963XLKBxNQecYxNbLFaWq67t3rFnKm9E8BAICi
4zUaE5J1tMVi7Vi9iks9Ml9SnNyZRQJKfQ+kaebHXbkyAaPmfv+26rqHKboA0uxA
nDOZVwXX45zBkp6g1sdHxJx8JLoGEnkC9eyvSi0C//tRLx86OhLErXwYcNkCf1it
VMRKrWYoXJTUNo6tRhvodM88UnnIo3u3CALjhgU4uC1RTMHV4ZCGBwiAOb8GozSl
s5YD1E1iKwEULloHnK6BIh6P5v8q7J6uf/xdqoKMjlWBHgq6/roxKvkSPA1DOZ3l
jTadcgKFnRUmc+JT9p/ZbCxkA/ALFg8++G+0ghECgYA8vG3M/utweLvq4RI7l7U7
b+i2BajfK2OmzNi/xugfeLjY6k2tfQGRuv6ppTjehtji2uvgDWkgjJUgPfZpir3I
RsVMUiFgloWGHETOy0Qvc5AwtqTJFLTD1Wza2uBilSVIEsg6Y83Gickh+ejOmEsY
6co17RFaAZHwGfCFFjO76Q==
-----END RSA PRIVATE KEY-----"""


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
          script {
            remote.name = eon_ip
            remote.host = eon_ip
          }
          sshCommand remote: remote, command: "echo /VERSION"
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
