pipeline {
  agent any

  stages {

    stage('Get Code') {
      agent any
      steps {
        git branch: 'master', changelog: false, poll: false, url: 'https://github.com/luiinge/unir-todo-list-aws.git'
      }
    }


    stage('Deploy') {
      agent any
      steps {
        sh 'sam build'
        sh 'sam deploy --config-env production --no-confirm-changeset --no-fail-on-empty-changeset --no-progressbar'
        // Recupera la URL from CloudFormation outputs y la stashea
        script {
          def baseURL = sh(script: "aws cloudformation describe-stacks --stack-name production-todo-list-aws --query 'Stacks[0].Outputs[?OutputKey==`BaseUrlApi`].OutputValue' --output text", returnStdout: true).trim()
          echo "Base URL: ${baseURL}"
          writeFile file: 'baseURL.txt', text: baseURL
        }
        stash includes: 'baseURL.txt', name: 'baseURL'
      }
    }

    stage ('Rest Test') {
      agent any
      steps {
        unstash 'baseURL'
        // solo ejecutar test de smoke
        sh '''
          export BASE_URL=$(cat baseURL.txt)
          echo "Testing API at $BASE_URL"
          pytest test/integration/todoApiTest.py -m smoke-v --junit-xml=results.xml
        '''
        junit 'results.xml'
      }
    }

    


  }
}
  
