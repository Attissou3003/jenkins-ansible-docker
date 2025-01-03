node {
   def registryProjet='registry.gitlab.com/attissou3003/presentations-jenkins/wartest'
   def IMAGE="${registryProjet}:version-${env.BUILD_ID}"
    stage('Build - Clone') {
          git 'https://github.com/Attissou3003/war-build-docker.git'
    }
    stage('Build - Maven package'){
            sh 'mvn package'
    }
    def img = stage('Build') {
          docker.build("$IMAGE",  '.')
    }
    stage('Build - Test') {
            img.withRun("--name run-$BUILD_ID -p 8082:8080") { c ->
            sh """ 
                docker ps -a -q --filter name=run-$BUILD_ID | xargs -r docker rm -f
            """
            sh "docker run -d --name run-$BUILD_ID -p 8082:8080 $IMAGE"
            sh 'docker ps'
            sh 'netstat -ntaup'
            sh 'sleep 30s'
            sh 'curl 127.0.0.1:8082'
            sh 'docker ps'
            sh """
                CONTAINER_ID=\$(docker ps -q -f name=run-$BUILD_ID)
                if [ -n "\$CONTAINER_ID" ]; then
                    docker stop run-$BUILD_ID
                    docker rm run-$BUILD_ID
                fi
            """
          }
    }
    stage('Build - Push') {
          docker.withRegistry('https://registry.gitlab.com', 'reg1') {
              img.push 'latest'
              img.push()
          }
    }
    stage('Deploy - Clone') {
          git 'https://github.com/Attissou3003/jenkins-ansible-docker.git'
    }
    stage('Deploy - End') {
      ansiblePlaybook (
          colorized: true,
          become: true,
          playbook: 'playbook.yml',
         inventory: '${HOST},',
          extras: "--extra-vars 'image=$IMAGE'"
      )
    }

}

