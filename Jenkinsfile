node {
    stage('Checkout') {
        checkout scm
    }
    stage('Verify Test Plan') {
        // List files to verify the presence of my_test_plan.jmx
        sh 'ls -la jmeter/my_test_plan.jmx'
    }
    stage('Build') {
        try {
            docker.image('maven:3.9.9').inside {
                sh 'mvn clean package'
            }
        } catch (Exception e) {
            echo "Build failed: ${e.getMessage()}"
            error("Build stage failed")
        }
    }
    stage('Test') {
        try {
            parallel (
                "unit-tests": {
                    docker.image('maven:3.9.9').inside {
                        sh 'mvn test'
                    }
                },
                "performance-tests": {
                    docker.image('justb4/jmeter:5.5').inside {
                        sh 'jmeter -n -t jmeter/my_test_plan.jmx -l result.jtl'
                    }
                    archiveArtifacts artifacts: 'result.jtl', allowEmptyArchive: true
                    perfReport sourceDataFiles: 'result.jtl'
                }
            )
        } catch (Exception e) {
            echo "Test stage failed: ${e.getMessage()}"
            error("Test stage failed")
        }
    }
    // stage('Deploy') {
    //     withCredentials([usernamePassword(credentialsId: 'my-ansible-credentials', usernameVariable: 'ANSIBLE_USER', passwordVariable: 'ANSIBLE_PASSWORD')]) {
    //         sh "ansible-playbook -i my-inventory -u ${ANSIBLE_USER} -k -v deploy.yml"
    //     }
    // }
}