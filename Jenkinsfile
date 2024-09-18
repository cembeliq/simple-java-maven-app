node {
    stage('Build') {
        sh 'mvn clean package'
    }
    stage('Test') {
        parallel (
            "unit-tests": {
                sh 'mvn test'
            },
            "performance-tests": {
                sh 'jmeter -n -t my_test_plan.jmx'
            }
        )
    }
    // stage('Deploy') {
    //     withCredentials([usernamePassword(credentialsId: 'my-ansible-credentials', usernameVariable: 'ANSIBLE_USER', passwordVariable: 'ANSIBLE_PASSWORD')]) {
    //         sh "ansible-playbook -i my-inventory -u ${ANSIBLE_USER} -k -v deploy.yml"
    //     }
    // }
}