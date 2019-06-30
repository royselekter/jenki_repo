node() {
def DockerImage = "webserver:1.2"

stage('Pre') { // Run pre-build steps
  cleanWs()
  sh "docker rm -f webserver || true"
}

stage('Git') { // Get code from GitLab repository
  git branch: 'master',
    url: 'https://github.com/royselekter/flask-http.git'
}

stage('Build') { // Run the docker build
  sh "docker build --tag ${DockerImage} ."
}
stage('Run') { // Run the built image
  sh "docker run -d --name webserver --rm -p 8081:5000 ${DockerImage}; sleep 5"
}

stage('Test') { // Run tests on container
  def dockerOutput = sh (
      script: 'curl http://localhost:8081/goaway',
      //script: 'curl http://localhost:8081/goaway',
      returnStdout: true
      ).trim()
//  sh "docker rm -f webserver"

  if ( dockerOutput == 'GO AWAY!' ) {
      currentBuild.result = 'SUCCESS'
  } else {
      currentBuild.result = 'FAILURE'
      sh "echo Webserver returned ${dockerOutput}"
  }
  return
}
stage('Push') { // Push the built image to docker hub
   withDockerRegistry(credentialsId: 'docker_hub') {
      sh "docker tag ${DockerImage} royselekter/${DockerImage}"
      sh "docker push royselekter/${DockerImage}"
   }
}

stage ('Deploy') { // deploy to k8s
  sshagent(credentials : ['ssh_jen']) {
     sh 'ssh -o StrictHostKeyChecking=no -i ~jenkins/.ssh/id_rsa ubuntu@192.168.10.40 ""kubectl apply -f /home/ubuntu/k8s_repo/k8s-pod.yml""'
   }
 }
}
