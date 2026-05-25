storeBaseURL() {

}
fetchBaseURL() {
  unstash 'baseURL'
  def baseURL = readFile('baseURL.txt').trim()
  echo "Fetched Base URL: ${baseURL}"
  return baseURL
}
pipeline {
  agent any

  stages {

    stage('Get Code') {
      agent any
      steps {
        git branch: 'develop', changelog: false, poll: false, url: 'https://github.com/luiinge/unir-todo-list-aws.git'
      }
    }

    stage('Static Test') {
      agent any
      steps {
        sh 'flake8 src --format=pylint --exit-zero > flake8.out'
        catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
          recordIssues tools: [flake8(pattern:'flake8.out')]
        }
        sh 'bandit -r src -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"'
        catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
          recordIssues tools: [pyLint(pattern:'bandit.out')]
        }
      }
    }

    stage('Deploy') {
      agent any
      steps {
        sh 'sam build'
        sh 'sam deploy --config-env staging --no-confirm-changeset --no-fail-on-empty-changeset --no-progressbar'
        // Recupera la URL from CloudFormation outputs y la stashea
        def baseURL = sh(script: "aws cloudformation describe-stacks --stack-name staging-todo-list --query 'Stacks[0].Outputs[?OutputKey==`TodoListApi`].OutputValue' --output text", returnStdout: true).trim()
        echo "Base URL: ${baseURL}"
        writeFile file: 'baseURL.txt', text: baseURL
        stash includes: 'baseURL.txt', name: 'baseURL'
      }
    }

    stage ('Rest Test') {
      agent any
      steps {
        unstash 'baseURL'
        sh '''
          export BASE_URL=$(cat baseURL.txt)
          echo "Testing API at $BASE_URL"
          pytest test/integration/todoApiTest.py -v --junit-xml=results.xml
        '''
        junit 'results.xml'
      }
    }

    stage('Promote') {
      agent any
      steps {
        echo "Promoting to production..."
      }
    }
  }
}