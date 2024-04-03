def createPackage(String architecture) {
    script {
        sh """
        mkdir -p build/usr/bin
        mv build/orchent-${architecture}-linux build/usr/bin/orchent
        fpm -s dir -t deb -n orchent -v 1.0.0 -C build/ \\
            -p orchent_1.0.0_${architecture}.deb .
        """
    }
}

pipeline {
  agent none
  stages {
        stage('Build') {
            agent {
                docker {
                  label 'jenkinsworker00'
                  image 'golang:1.16.15'
                  reuseNode true
                }
            }
            steps {
                script {
                  sh '''
                  mkdir -p build
                  export GOCACHE=$WORKSPACE/.cache/go-build
                  CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -ldflags ' -w -extldflags "-static"' -ldflags "-B 0x$(head -c20 /dev/urandom|od -An -tx1|tr -d ' \n')" -o build/orchent-amd64-linux orchent.go
                  CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -a -ldflags ' -w -extldflags "-static"' -ldflags "-B 0x$(head -c20 /dev/urandom|od -An -tx1|tr -d ' \n')" -o build/orchent-arm64-linux orchent.go
                  CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build -a -ldflags '-w -extldflags "-static"' -o build/orchent-amd64-darwin orchent.go
                  CGO_ENABLED=0 GOOS=darwin GOARCH=arm64 go build -a -ldflags '-w -extldflags "-static"' -o build/orchent-arm64-darwin orchent.go
                  '''
                }  
            }
        }
        
        stage('Package') {
            agent {
                docker {
                    label 'jenkinsworker00'
                    image 'marica/fpm:latest'
                    reuseNode true
                }
            }
            steps {
                
                    createPackage('amd64')
                    createPackage('arm64')
                
            }
            post {
               always {
                 cleanWs()
               }
             }
        }
        stage('Upload to Nexus'){
          steps{
            nexusArtifactUploader (
              nexusVersion: 'nexus3',
              protocol: 'https',
              nexusUrl: 'repo.cnaf.cloud.infn.it',
              version: '1.0.0',
              repository: 'orchent',
              credentialsId: 'nexus-credentials',
              artifacts [ 
                  [ artifactId: 'orchent-amd64', type: 'deb', classifier: '', file: 'orchent_1.0.0_amd64.deb' ],
                  [ artifactId: 'orchent-arm64', type: 'deb', classifier: '', file: 'orchent_1.0.0_arm64.deb' ]
              ]
            )
             
            }
          }
  }
}
