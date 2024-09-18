node {
    stage('Build') {
        docker.image('maven:3.9.9').inside {
            sh 'mvn clean package'
        }
    }
    stage('Test') {
        parallel (
            "unit-tests": {
                docker.image('maven:3.9.9').inside {
                    sh 'mvn test'
            }
            },
            "performance-tests": {
                docker.image('jmeter:latest').inside {
                    sh 'jmeter -n -t my_test_plan.jmx -l result.jtl'
                }
                archiveArtifacts artifacts: 'result.jtl', allowEmptyArchive: true
                perfReport sourceDataFiles: 'result.jtl'
            }
        )
    }
    // stage('Deploy') {
    //     withCredentials([usernamePassword(credentialsId: 'my-ansible-credentials', usernameVariable: 'ANSIBLE_USER', passwordVariable: 'ANSIBLE_PASSWORD')]) {
    //         sh "ansible-playbook -i my-inventory -u ${ANSIBLE_USER} -k -v deploy.yml"
    //     }
    // }
}