node {
    checkout scm
    
    def SF_USERNAME = 'arpankhadka88@cunning-panda-o148fn.com'
    def SF_INSTANCE_URL = "https://login.salesforce.com"
    def PACKAGE_NAME = 'FirstPackage'
    
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
        
        stage('Assign Permission Set') {
            script {
                def scratchOrgAlias = 'PLTest'
                
                echo "Finding permission sets from source code..."
                
                def permSetName = sh(
                    script: """
                      find force-app -name '*.permissionset-meta.xml' -type f | \
                      head -1 | \
                      xargs basename | \
                      sed 's/.permissionset-meta.xml//'
                    """,
                    returnStdout: true
                ).trim()
                
                if (permSetName) {
                    echo "Found permission set: ${permSetName}"
                    echo "Assigning permission set ${permSetName} to admin user..."
                    sh """
                      sf org assign permset \
                        --name ${permSetName} \
                        --target-org ${scratchOrgAlias}
                    """
                    echo "Permission set ${permSetName} assigned successfully"
                } else {
                    echo "No custom permission sets found in source. Skipping assignment."
                }
            }
        }
        
        stage('Run Unit Tests') {
            script {
                def scratchOrgAlias = 'PLTest'
                def testLevel = 'RunLocalTests'
                
                echo "=========================================="
                echo "Running Apex unit tests in ${scratchOrgAlias}..."
                echo "Test Level: ${testLevel}"
                echo "=========================================="
                
                // Check if there are any test classes
                def testClassCount = sh(
                    script: "find force-app -name '*Test*.cls' -o -name '*_Test.cls' | wc -l",
                    returnStdout: true
                ).trim()
                
                if (testClassCount.toInteger() > 0) {
                    def testStatus = sh(
                        script: """
                          sf apex run test \
                            --target-org ${scratchOrgAlias} \
                            --wait 10 \
                            --result-format human \
                            --code-coverage \
                            --test-level ${testLevel}
                        """,
                        returnStatus: true
                    )
                    
                    if (testStatus != 0) {
                        error "Salesforce unit test run failed in ${scratchOrgAlias}"
                    } else {
                        echo "=========================================="
                        echo "All unit tests passed successfully!"
                        echo "=========================================="
                    }
                } else {
                    echo "=========================================="
                    echo "No Apex test classes found. Skipping unit tests."
                    echo "=========================================="
                }
            }
        }
        
        stage('Create Package Version') {
            script {
                echo "=========================================="
                echo "Creating package version for ${PACKAGE_NAME}..."
                echo "=========================================="
                
                sh """
                  sf package version create \
                    --package ${PACKAGE_NAME} \
                    --installation-key-bypass \
                    --wait 10 \
                    --target-dev-hub ${SF_USERNAME} \
                    --json
                """ 
                
                echo "=========================================="
                echo "Package version created successfully!"
                echo "Check the output above for the Package Version ID"
                echo "=========================================="
            }
        }
    }
}
