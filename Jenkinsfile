pipeline {
    agent any

    options {
        timestamps()
    }

    environment {
        APP_NAME = 'nginx-demo'
        APP_NAMESPACE = 'virenaz-dev'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'GitHub source code checked out successfully.'
            }
        }

        stage('Validate Files') {
            steps {
                sh '''
                    set -eu
                    test -s index.html
                    test -s openshift/deployment.yaml
                    test -s openshift/service-route.yaml
                    grep -q 'NGINX deployed successfully' index.html
                    echo "Application files validated."
                '''
            }
        }

        stage('Prepare Website ConfigMap') {
            steps {
                sh '''
                    set -eu

                    oc project "$APP_NAMESPACE"

                    oc create configmap nginx-demo-html \
                      --from-file=index.html=index.html \
                      --dry-run=client -o yaml | oc apply -f -
                '''
            }
        }

        stage('Deploy to OpenShift') {
            steps {
                sh '''
                    set -eu

                    oc apply -f openshift/deployment.yaml
                    oc apply -f openshift/service-route.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eu

                    oc rollout status deployment/"$APP_NAME" \
                      -n "$APP_NAMESPACE" --timeout=180s

                    oc get deployment,service,route \
                      -n "$APP_NAMESPACE"

                    echo "Application deployment completed."
                    echo "Application URL:"
                    oc get route "$APP_NAME" -n "$APP_NAMESPACE" \
                      -o jsonpath='https://{.spec.host}{"\\n"}'
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: NGINX deployment pipeline completed.'
        }
        failure {
            echo 'FAILED: Review the failed stage and its console logs.'
        }
    }
}
