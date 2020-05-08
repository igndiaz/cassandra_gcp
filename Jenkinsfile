def CASSANDRA_IP
def CASSANDRA_NETWORK
pipeline {
    agent any 
    parameters {
        string(name: 'CLUSTER_NAME', defaultValue: 'Test Cluster', description: 'Nombre del Cluster Cassandra')
        choice(name: 'ENV', choices: ['PROD', 'DEV'], description: 'Ambiente')
        string(name: 'NODOS', defaultValue: '1', description: 'Cantidad de Nodos Cluster')
        choice(name: 'SNITCH', choices: ['SimpleSnitch', 'GossipingPropertyFileSnitch', 'GoogleCloudSnitch'], description: 'Tipo de Snitch Cluster')
    }
    stages {
        stage ('Creación Máquinas') {
            steps {
            sh "gcloud config set project my-own-project-252421"
                script {
                    for (loopIndex=0; loopIndex < Integer.parseInt("${params.NODOS}");loopIndex++) {
               sh "gcloud beta compute --project=my-own-project-252421 instances create cassandra-dev-${loopIndex} --zone=us-central1-a --machine-type=n1-standard-8 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=812385867631-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server --image=debian-9-stretch-v20200420 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=cassandra-dev-${loopIndex} --create-disk=mode=rw,size=100,type=projects/my-own-project-252421/zones/us-central1-a/diskTypes/pd-ssd,name=cassandra-dev-disk-${loopIndex},device-name=cassandra-dev-disk-${loopIndex} --reservation-affinity=any" 
                }
          for (loopIndex=0; loopIndex < Integer.parseInt("${params.NODOS}");loopIndex++) {
              sh """
              CASSANDRA_IP=\$(gcloud compute instances describe cassandra-dev-${loopIndex} --zone=us-central1-a --format='value(networkInterfaces.networkIP)')
              echo \$CASSANDRA_IP
              """

          }
                       }   
            } 
        }
        stage('Instalacion Cassandra') {
            steps {
                script {
                for (loopIndex=0; loopIndex < Integer.parseInt("${params.NODOS}");loopIndex++){
                sh """ 
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command 'sudo apt-get install -y apt-transport-https'
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command 'sudo apt-get update'
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command 'echo "deb https://downloads.apache.org/cassandra/debian 311x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list'
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command 'curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -'
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command 'sudo apt-get update'
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command 'sudo apt-get -y install cassandra'
                """
                }
                       }
            }  
        }
        stage('Modificaciones Nodo') {
            steps {
                script {
                for (loopIndex=0; loopIndex < Integer.parseInt("${params.NODOS}");loopIndex++){
                sh """
                CASSANDRA_NETWORK=\$(gcloud compute instances describe cassandra-dev-${loopIndex} --zone=us-central1-a --format='value(networkInterfaces.networkIP)')
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command "sudo sed -i 's/localhost/\$CASSANDRA_NETWORK/gI' /etc/cassandra/cassandra.yaml"
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command "sudo sed -i 's/Test Cluster/${params.CLUSTER_NAME}/gI' /etc/cassandra/cassandra.yaml"
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command "sudo sed -i 's/SimpleSnitch/${params.SNITCH}/gI' /etc/cassandra/cassandra.yaml"
                """
                }
            }
            }
        }  
        stage('Inicio de Servicio & Validación') {
            steps {
                 script {
                for (loopIndex=0; loopIndex < Integer.parseInt("${params.NODOS}");loopIndex++){
                sh """ 
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command "sudo service cassandra start"
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command "sleep 30"
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command "nodetool status"
                gcloud compute ssh cassandra-dev-${loopIndex} --zone=us-central1-a --command "cqlsh -e 'describe keyspaces;'"
                """
                }
                }  
            }
        }    
    }
}
