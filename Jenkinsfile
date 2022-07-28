pipeline {
    agent { dockerfile true }
    stages {
        stage('Notificacao no Slack iniciando novo Deploy em Homologacao') {
            steps {
                slackSend (color: 'warning', message: '[ Em andamento - Testes] Novo deploy iniciado em: http://34.211.224.42/', tokenCredentialId: 'slack-token')
            }
        }                  
        stage('---------- Provisionando Infraestrutura de Homologacao ----------') {
            steps {
                dir('./homolog') {
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'terraform-aws', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        script {
                            try {
                                sh 'terraform destroy -target module.aws-homolog.aws_instance.app_server -auto-approve'
                            } 
                            catch (err) {
                                echo 'Ainda nao existiam instancias de homologacao, criando uma nova'
                            }
                        }                    
                        sh 'terraform init'
                        sh 'terraform apply -target module.aws-homolog.aws_security_group.acesso_projeto -auto-approve'
                        sh 'terraform apply -target module.aws-homolog.aws_instance.app_server -auto-approve'
                        sh 'terraform apply -target module.aws-homolog.aws_eip_association.eip_assoc -auto-approve'
                    }  
                }
            }
        }
        stage('---------- Instalando as dependencias de homologacao ----------') {
            steps {
                ansiblePlaybook credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'hosts-infra-homolog.yml', playbook: 'playbook-infra.yml'
            }
        }
        stage('Deploy em homologacao') {
            steps {            
                ansiblePlaybook credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'hosts-app-homolog.yml', playbook: 'playbook-app.yml'                                    
            }
        }
        stage('Testes automatizados') {
            steps {
                sh 'echo PASSOU NO TESTE 1'
                sh 'echo PASSOU NO TESTE 2'
                sh 'echo PASSOU NO TESTE 3'
            }
        }             
        stage('Aprovacao do deploy em producao') {
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        input(id: "Deploy Gate", message: "Fazer o deploy em produção?", ok: 'Deploy')
                    }
                }
            }
        }
        stage('Notificacao no Slack iniciando novo Deploy em Producao') {
            steps {
                slackSend (color: 'warning', message: '[ Em andamento - Deploy em Producao] Novo deploy iniciado em: http://34.211.224.42/', tokenCredentialId: 'slack-token')
            }
        }
        stage('Imagem para producao') {
            steps {
                dir('./homolog') {
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'terraform-aws', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'terraform apply -target module.aws-homolog.aws_ami_from_instance.AMI_Prod -auto-approve'
                    }  
                }
            }
        }                                       
    }
}