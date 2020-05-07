pipeline {
    agent any 
    parameters {
        string(name: 'CLUSTER_NAME', defaultValue: 'Test Cluster', description: 'Nombre del Cluster Cassandra')
        integer(name: 'NODOS', defaultValue: '1', description: 'Cantidad de Nodos Cluster')
        string(name: 'SNITCH', defaultValue: 'SimpleSnitch', description: 'Tipo de Snitch Cluster')
    }
    stages {
        stage ('Creación Máquinas') {
            sh "gcloud config set project my-own-project-252421"
            for (int loopIndex=0; loopIndex < ${params.NODOS},loopIndex++) {
               sh "gcloud beta compute --project=my-own-project-252421 instances create cassandra-dev-${loopIndex} --zone=us-central1-a --machine-type=custom-16-32768 --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=812385867631-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server,https-server --image=debian-9-stretch-v20200420 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=cassandra-dev-${loopIndex} --create-disk=mode=rw,size=100,type=projects/my-own-project-252421/zones/us-central1-a/diskTypes/pd-ssd,name=cassandra-dev-disk-1,device-name=cassandra-dev-disk-${loopIndex} --reservation-affinity=any" 
            }
            for (int loopIndex=0; loopIndex < ${params.NODOS},loopIndex++) {
           sh "CASSANDRA_IP_${loopIndex}=$(gcloud compute instances describe cassandra-dev-${loopIndex} --zone=us-central1-a --format='value(networkInterfaces[0].networkIP)')"
        }

        }
        stage('Instalacion Cassandra') {
            steps {
                for (int loopIndex=0; loopIndex < ${params.NODOS},loopIndex++){
                sh "gcloud compute ssh cassandra-dev-${loopIndex}"
                sh "echo 'deb https://downloads.apache.org/cassandra/debian 311x main' | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list"
                sh "curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -"
                sh "sudo apt-get -y install cassandra"
                sh "exit"
                }

            }  
        }
        stage('Modificaciones Nodo') {
            steps {
                sh "sudo sed -i 's/Test Cluster/${params.CLUSTER_NAME}/gI' /etc/cassandra/cassandra.yaml"
            }   
        }  
        stage('Inicio de Servicio & Validación') {
            steps {
                sh "sudo service cassandra start"
                sh "sleep 30"
                sh "nodetool status"
                sh "cqlsh -e 'describe keyspaces;'"
            }  
        }    
    }
}