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
        
        stage('Delete Existing Scratch Org') {
            script {
                def scratchOrgAlias = 'PLTest'
                
                // Check if scratch org exists
                def orgExists = sh(
                    script: "sf org list --json | grep -q '\"alias\":\"${scratchOrgAlias}\"'",
                    returnStatus: true
                )
                
                if (orgExists == 0) {
                    echo "Scratch org with alias ${scratchOrgAlias} exists. Deleting..."
                    sh """
                      sf org delete scratch --target-org ${scratchOrgAlias} --no-prompt
                    """
                    echo "Scratch org ${scratchOrgAlias} deleted successfully"
                } else {
                    echo "No existing scratch org with alias ${scratchOrgAlias} found. Proceeding to create new one."
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
    }
}
