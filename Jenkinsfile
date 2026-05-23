pipeline {
     agent any
     
     stages {
       stage('Get Code') {
         steps {
           git branch: 'develop', url: 'https://github.com/TU_USUARIO/todo-list-aws.git'
         }
       }
       
       stage('Static Analysis') {
         steps {
           sh 'pip install flake8 bandit'
           sh 'flake8 src/'
           sh 'bandit -r src/'
         }
       }
       
       stage('Deploy to Staging') {
         steps {
           sh 'sam build'
           sh 'sam deploy --config-file samconfig.toml --config-env staging'
           // Capturar la URL
           sh 'aws cloudformation describe-stacks --stack-name todo-staging --query "Stacks[0].Outputs[0].OutputValue" > url.txt'
           sh 'cat url.txt' // Para verlo en logs
         }
       }
       
       stage('REST Tests') {
         steps {
           sh 'pip install pytest requests'
           sh 'BASE_URL=$(cat url.txt) pytest tests/test_api.py -v'
         }
       }
       
       stage('Promote to Master') {
         steps {
           // Esto requiere credenciales GitHub y permisos
           sh '''
             git config user.email "jenkins@example.com"
             git config user.name "Jenkins"
             git checkout master
             git merge develop
             git push origin master
           '''
         }
       }
     }
   }
