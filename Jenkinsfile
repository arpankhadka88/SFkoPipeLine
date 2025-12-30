node {
    checkout scm
    
    def SF_USERNAME = 'arpankhadka88@cunning-panda-o148fn.com'
    def SF_INSTANCE_URL = "https://login.salesforce.com"
    
    withCredentials([
        file(credentialsId: 'PLKoKey', variable: 'JWT_KEY_FILE'),
        string(credentialsId: 'PLKoConsumerID', variable: 'CQ_CONSUMER_SECRET')
    ]) {
        stage('Authorize DevHub') {
            sh """
              sf org login jwt \
                --instance-url ${SF_INSTANCE_URL} \
                --jwt-key-file ${JWT_KEY_FILE} \
                --client-id ${CQ_CONSUMER_SECRET} \
                --username ${SF_USERNAME}
            """
        }
        
        stage('List Existing Scratch Orgs') {
            echo "=========================================="
            echo "Listing all existing scratch orgs:"
            echo "=========================================="
            sh """
              sf org list
            """
            echo "=========================================="
        }
        
        stage('Delete Existing Scratch Org') {
            script {
                def scratchOrgAlias = 'PLTest'
                
                echo "Attempting to delete scratch org ${scratchOrgAlias} if it exists..."
                def deleteStatus = sh(
                    script: "sf org delete scratch --target-org ${scratchOrgAlias} --no-prompt",
                    returnStatus: true
                )
                
                if (deleteStatus == 0) {
                    echo "Scratch org ${scratchOrgAlias} deleted successfully"
                    echo "Waiting for deletion to propagate..."
                    sleep(time: 5, unit: 'SECONDS')
                } else {
                    echo "No scratch org ${scratchOrgAlias} found to delete, or deletion failed. Proceeding..."
                }
            }
        }
        
        stage('Create Test Scratch Org') {
            script {
                def scratchOrgAlias = 'PLTest'
                
                echo "Creating new scratch org with alias ${scratchOrgAlias}..."
                sh """
                  sf org create scratch \
                    --definition-file config/project-scratch-def.json \
                    --alias ${scratchOrgAlias} \
                    --set-default \
                    --duration-days 7 \
                    --target-dev-hub ${SF_USERNAME}
                """
                echo "Scratch org ${scratchOrgAlias} created successfully"
            }
        }
        
        stage('Generate Scratch Org Password') {
            script {
                def scratchOrgAlias = 'PLTest'
                
                echo "Generating password for scratch org ${scratchOrgAlias}..."
                sh """
                  sf org generate password --target-org ${scratchOrgAlias}
                """
                
                echo "=========================================="
                echo "Scratch Org Credentials:"
                echo "=========================================="
                sh """
                  sf org display --target-org ${scratchOrgAlias} --verbose
                """
                echo "=========================================="
                echo "Save these credentials to login from your local machine!"
                echo "=========================================="
            }
        }
        
        stage('Deploy to PLTest Scratch Org') {
            script {
                def scratchOrgAlias = 'PLTest'
                
                echo "Deploying source to scratch org ${scratchOrgAlias}..."
                sh """
                  sf project deploy start \
                    --source-dir force-app \
                    --target-org ${scratchOrgAlias}
                """
                echo "Deployment to ${scratchOrgAlias} completed successfully"
            }
        }
    }
}
