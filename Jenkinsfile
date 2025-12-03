pipeline {
  agent any

  options {
    // Optional: avoids Jenkins doing an implicit checkout in some situations
    skipDefaultCheckout(true)
  }

  stages {
    stage('Checkout') {
      steps {
        // Ensure checkout happens in the *current* workspace used by later stages
        checkout([$class: 'GitSCM',
          branches: [[name: '*/main']],
          userRemoteConfigs: [[
            url: 'https://github.com/eyalhart-2025/learn-jenkins-app.git',
            credentialsId: 'github-creds'
          ]]
        ])
        sh 'ls -la'  // Verify files are here (should include package.json)
      }
    }

    stage('Build') {
      steps {
        // Run Node build inside container, mounting THIS workspace path
        withDockerContainer(image: 'node:18-alpine') {
          sh '''
            node --version
            npm --version
            ls -la
            test -f package.json || (echo "package.json missing"; exit 1)
            npm ci
            npm run build
          '''
        }
      }
    }
    stage('Test') {
        agent {
            docker {
                image 'node:18-alpine'
                reuseNode true
            }
        }
        steps {
            sh '''
                test -f build/index.html
                npm test
            '''
        }
    }

    stage('E2E') {
        agent {
            docker {
                image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                reuseNode true
            }
        }
        steps {
            sh '''
                npm install -g serve
                serve -s build
                npx playwright test
            '''
        }
    }
  }
  
  post {
      always {
          junit 'test-results/junit.xml'
      }
  }
}