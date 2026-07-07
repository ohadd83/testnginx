pipeline {

    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: kubectl-agent
spec:
  serviceAccountName: jenkins

  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
'''
        }
    }

    stages {
        stage('Test') {
            steps {
                container('kubectl') {
                    sh 'kubectl get nodes'
                }
            }
        }
    }
}

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Kubernetes') {
            steps {
                sh '''
                    kubectl cluster-info
                    kubectl get nodes
                '''
            }
        }

        stage('Create NGINX Pod') {
            steps {
                sh '''
                    kubectl apply -f nginx-pod.yaml
                    kubectl wait --for=condition=Ready pod/nginx-demo --timeout=120s
                '''
            }
        }

        stage('Keep Pod Running') {
            steps {
                echo 'Keeping the pod alive for 60 seconds...'
                sh 'sleep 60'
            }
        }

        stage('Delete Pod') {
            steps {
                sh '''
                    kubectl delete -f nginx-pod.yaml --ignore-not-found=true
                '''
            }
        }
    }

    post {
        always {
            sh 'kubectl delete -f nginx-pod.yaml --ignore-not-found=true || true'
        }
    }
}
