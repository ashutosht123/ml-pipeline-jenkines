pipeline {
    agent any
    environment {
        EC2_HOST = credentials('EC2_HOST')   // Store EC2_HOST in Jenkins credentials
        EC2_USER = credentials('EC2_USER')   // Store EC2_USER in Jenkins credentials
        SSH_KEY = credentials('SSH_KEY')     // Store SSH_KEY in Jenkins credentials
    }
    stages {
        stage('Checkout Repository') {
            steps {
                checkout scm  // Check out the repository
            }
        }
        stage('Set Up Python') {
            steps {
                bat '''
                    python -m venv myenv    # Create a virtual environment
                    call myenv\\Scripts\\activate
                    python --version        # Verify Python version
                '''
            }
        }
        stage('Install Dependencies') {
            steps {
                bat '''
                    call myenv\\Scripts\\activate
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        stage('Check Dataset Changes and Retrain') {
            steps {
                bat '''
                    call myenv\\Scripts\\activate
                    if exist train.py (
                        python train.py  # Retrain the model if dataset changes
                    ) else (
                        echo train.py not found, skipping training step.
                    )
                '''
            }
        }
        stage('Deploy Model to EC2') {
            steps {
                bat '''
                    REM Securely store SSH key and set permissions
                    echo %SSH_KEY% > ec2_key
                    chmod 600 ec2_key

                    REM Ensure EC2 is in known hosts
                    mkdir .ssh 2>nul
                    ssh-keyscan -H %EC2_HOST% >> .ssh\\known_hosts

                    REM Create folder on EC2 if it doesn't exist
                    ssh -i ec2_key %EC2_USER%@%EC2_HOST% "mkdir -p /home/%EC2_USER%/models"

                    REM Copy model file to EC2
                    scp -i ec2_key models\\model_*.pkl %EC2_USER%@%EC2_HOST%:/home/%EC2_USER%/models/
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
