node('jenkins-slave') {
    stage('Clone') {
      echo "1.Clone Stage"
    //  git url: "https://github.com/peter-wins/jenkins-demo.git"
      checkout scm
      script {
        build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
        if (env.BRANCH_NAME != 'master') {
          build_tag = "${env.BRANCH_NAME}-${build_tag}"
        }
      }
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
      echo "3.Build Docker Image Stage"
      sh "docker build -t devopspeter/jenkins-demo:${build_tag} ."
    }
    stage('Push') {
      echo "4.Push Docker Image Stage"
      withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
        sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword}"
        sh "docker push devopspeter/jenkins-demo:${build_tag}"
      }
    }
    stage('Deploy') {
      echo "5. Deploy Stage"
    //  def userInput = input(
    //      id: 'userInput',
    //      message: 'Choose a deply environment',
    //      parameters: [
    //          [
    //              $class: 'ChoiceParameterDefinition',
    //              choices: "Dev\nQA\nProd",
    //              name: 'Env'
    //          ]
    //      ]
    //  )
    //  echo "This is a deploy step to ${userInput}"
    if (env.BRANCH_NAME == 'master') {
      input "确认要部署线上环境吗？"
    }
      sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
      sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"

    //  if (userInput == "Dev") {
    //      // deploy dev stuff
    //  }else if (userInput == "QA") {
    //      // deploy qa stuff
    //  }else {
    //      // deploy prod stuff
    //  }
      sh "kubectl apply -f k8s.yaml --record"
    }
}