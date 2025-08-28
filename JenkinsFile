pipeline {
  agent { label 'docker' } 

  stages {
    stage('Checkout') {
      steps {
        checkout scm 
      }
    }

    stage('Build/Compile') {
      steps {
        // 
        //
        // 
        sh 'npm ci'
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t webapp:ci .'
      }
    }

    stage('Deploy (Docker run)') {
      steps {
        sh '''
          docker rm -f web || true
          docker run -d --name web -p 8080:8080 webapp:ci
        '''
      }
    }

    stage('Healthcheck') {
      steps {
        sh '''
          for i in $(seq 1 30); do
            curl -fsS http://localhost:8080/health && exit 0
            sleep 2
          done
          echo "Service not healthy" >&2
          docker logs web || true
          exit 1
        '''
      }
    }

    stage('UI Tests (Playwright)') {
      steps {
        sh '''
          npx playwright install --with-deps
          npx playwright test
        '''
      }
    }
  }

  post {
    always {
      sh 'docker stop web || true'
      archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
    }
  }
}
