pipeline {
  agent any

  stages {
    stage('Initial cleanup') {
      steps {
        cleanWs()
      }
    }

    stage('Checkout SCM') {
      steps {
        git branch: 'main', url: 'https://github.com/princemaxi/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
        sh '''
          mv .env.sample .env
          composer install
          php artisan migrate
          php artisan db:seed
          php artisan key:generate
        '''
      }
    }

    stage('Execute Unit Tests') {
      steps {
        sh './vendor/bin/phpunit'
      }
    }

    stage('Code Analysis') {
      steps {
        sh '''
          mkdir -p build/logs
          phploc app/ --log-csv build/logs/phploc.csv
        '''
      }
    }

    stage('Package Artifact') {
      steps {
        sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
      }
    }

    stage('Upload Artifact to Artifactory') {
      steps {
        script {
          def server = Artifactory.server 'artifactory-server'
          def uploadSpec = """{
            "files": [
              {
                "pattern": "php-todo.zip",
                "target": "php-todo-repo/php-todo/",
                "props": "type=zip;status=ready"
              }
            ]
          }"""
          server.upload spec: uploadSpec
        }
      }
    }

    stage('Deploy to Dev Environment') {
      steps {
        build job: 'ansible-project/main',
          parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']],
          propagate: false,
          wait: true
      }
    }
  }
}

