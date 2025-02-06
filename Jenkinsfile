pipeline {
    agent any
    environment {
        EC2_HOST = credentials('EC2_HOST')  // EC2 Public IP stored in Jenkins credentials
        EC2_USER = credentials('EC2_USER')  // EC2 User stored in Jenkins credentials
        SSH_KEY = credentials('SSH_KEY')    // Private SSH key stored in Jenkins credentials
    }
    stages {
        stage('Checkout Repository') {
            steps {
                git branch: 'main', credentialsId: 'your-git-credentials-id', url: 'git@github.com:your-repo.git'
            }
        }
        
        stage('Set Up Python') {
            steps {
                sh '''
                    python3 -m venv myenv
                    echo "source myenv/bin/activate" >> ~/.bashrc
                '''
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '''
                    source myenv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Check Dataset Changes and Retrain') {
            steps {
                sh '''
                    source myenv/bin/activate
                    python train.py  # Retrain the model
                '''
            }
        }

        stage('Deploy Model to EC2') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'SSH_KEY', keyFileVariable: 'SSH_KEY_FILE')]) {
                        sh '''
                            # Ensure SSH key has correct permissions
                            chmod 600 $SSH_KEY_FILE

                            # Create .ssh directory if not exists
                            mkdir -p ~/.ssh

                            # Add EC2 host to known_hosts to avoid SSH prompt issues
                            ssh-keyscan -H $EC2_HOST >> ~/.ssh/known_hosts

                            # Ensure models directory exists on EC2
                            ssh -i $SSH_KEY_FILE $EC2_USER@$EC2_HOST '[ -d "/home/$EC2_USER/models" ] || mkdir -p /home/$EC2_USER/models'

                            # Copy trained model to EC2
                            scp -i $SSH_KEY_FILE models/model_*.pkl $EC2_USER@$EC2_HOST:/home/$EC2_USER/models/

                            # Restart service if needed (modify based on your deployment setup)
                            ssh -i $SSH_KEY_FILE $EC2_USER@$EC2_HOST "sudo systemctl restart model-service"
                        '''
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Model retrained and deployed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check the logs.'
        }
    }
}
