pipeline {
	
    agent {
        label "jenkins-maven"
    }
    
    environment {
      RELEASE_BRANCH        = "master"
      ORG                   = "introproventures"
      APP_NAME              = "aps"
      CHARTMUSEUM_CREDS     = credentials("jenkins-x-chartmuseum")
    }
    stages {
      stage("CI Build and push snapshot") {
        when {
          branch "PR-*"
        }
        environment {
          PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
          PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
          HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
        }
        steps {
          container("maven") {
            dir("preview") {
              sh "make preview"

              sh "jx preview"
            }
          }
        }
      }
      stage("Build Release") {
        when {
          branch "$RELEASE_BRANCH"
        }
        steps {
          // Working directory name must match with chart name
          container("maven") {
            dir("$APP_NAME") {
              // ensure we're not on a detached head
              sh "make checkout"
  
              // so we can retrieve the version in later steps
              sh "make next-version"
              
              // Let's build first
              sh "make build"

              // Let's test chart
              sh "make test"

              // Let's make tag in Git
              sh "make tag"

              // Publish release to Github
              sh "make changelog"
              
              // Release Helm chart in Chartmuseum  
              sh "make release"
            }
          }
        }
      }
      stage("Promote to Environments") {
        when {
          branch "$RELEASE_BRANCH"
        }
        environment {
          GITHUB_CHARTS_REPO    = "https://github.com/${ORG}/${APP_NAME}.git"
          PROMOTE_HELM_REPO_URL = "https://${ORG}.github.io/${APP_NAME}"
        }
        steps {
          container("maven") {
            dir("$APP_NAME") {
              // Let's publish in Github Charts Repo
              sh "make github"

              // Let's promote to environments 
              sh "make promote"
            }
          }
        }
      }
    }
    post {
        always {
            cleanWs()
        }
    }
}
