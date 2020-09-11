# LAB:  Elastic Google Cloud Infrastructure: Configuring an HTTP Load Balancer with Autoscaling

## Objectives:

In this lab you learn how to perform the following tasks:

  - Create a health check firewall rule

  - Create a NAT configuration using Cloud Router

  - Create a custom image for a web server

  - Create an instance template based on the custom image

  - Create two managed instance groups

  - Configure an HTTP load balancer with IPv4 and IPv6
  
  - Stress test an HTTP load balancer

## Steps:

1. Configure a health check firewall rule

   1. Create a firewall rule to allow health checks:

             gcloud compute firewall-rules list

             gcloud compute firewall-rules create fw-allow-health-checks --network default --priority 1000 --direction INGRESS --action ALLOW --target-tags allow-health-checks --source-ranges 130.211.0.0/22, 35.191.0.0/16 --rules tcp:22

2. Create a NAT configuration using Cloud Router

   1. Create the Cloud Router instance

             gcloud compute routers nats create nat-config --router=nat-router-us-central1 --auto-allocate-nat-external-ips --nat-all-subnet-ip-ranges --enable-logging --region us-central1

3. Create a custom image for a web server

   1. Create a VM:

             gcloud compute instances create webserver --zone us-central1-a --machine-type f1-micro --network default --subnet deafult --no-address --tags allow-health-checks

   2. Customize the VM:

      - Install Apache2:

             gcloud compute ssh webserver

             sudo apt-get update

             sudo apt-get install -y apache2

      - Start the Apache server:

             sudo service apache2 start

      - Test the default page for the Apache2 server:

             curl localhost

   3. Set the Apache service to start at boot:

        - In the webserver SSH terminal, set the service to start on boot:

              sudo update-rc.d apache2 enable

        - In the Cloud Console, select webserver, and then click Reset:

              gcloud compute instances reset webserver

        - Check the server by connecting via SSH to the VM and entering the following command:

              sudo service apache2 status

   4. Prepare the disk to create a custom image:

        - On the VM instances page, click webserver to view the VM instance details:

              gcloud compute instances describe webserver

        - Return to the VM instances page, click webserver, and click Delete:

              gcloud compute instances delete webserver

        - In the left pane, click Disks and verify that the webserver disk exists:

              gcloud compute disks list

   5. Create the custom image:

              gcloud compute images create mywebserver --source-image webserver

4. Configure an instance template and create instance groups:

   1. Configure the instance template:

              gcloud compute instance-templates create mywebserver-template --machine-type f1-micro --image mywebserver --tags allow-health-checks --no-address

   2. Create the managed instance groups:

        - Create a managed instance group in us-central1 and one in europe-west1:

              gcloud compute health-checks create tcp http-health-check --timeout 5 --check-interval 10 --unhealthy-threshold 3 --healthy-threshold 2 --port 80

              gcloud compute instance-groups managed create us-central1-mig --template mywebserver-template --zones=us-central1-b,us-central1-c,us-central1-f --health-check=http-health-check --initial-delay=60

              gcloud compute instance-groups managed set-autoscaling us-central1-mig --region us-central1 --cool-down-period 60 --max-num-replicas 2 --min-num-replicas 1 --target-load-balancing-utilization 0.8 --mode on

        - Repeat the same procedure for europe-west1-mig in europe-west1:

              gcloud compute instance-groups managed create europe-west1-mig --template mywebserver-template --zones=europe-west1-b,europe-west1-c,europe-west1-d --health-check=http-health-check --initial-delay=60

              gcloud compute instance-groups managed set-autoscaling europe-west1-mig --region europe-west1 --cool-down-period 60 --max-num-replicas 2 --min-num-replicas 1 --target-load-balancing-utilization 0.8 --mode on

   3. Verify the backends:

              gcloud compute instances list

5. Configure the HTTP load balancer

   1. Configure the backend:

              gcloud compute backend-services create http-backend --protocol=HTTP --port-name=http --health-checks=http-health-check --global

              gcloud compute backend-services add-backend http-backend --instance-group us-central1-mig --instance-group-zone=us-central1-b --global --port 80

              gcloud compute backend-services add-backend http-backend --instance-group europe-west1-mig --instance-group-zone=europe-west1-mig --global --port 80

              gcloud compute url-maps create web-map-https --default-service http-backend

              gcloud compute forwarding-rules create fr-lb-1 --region=us-central1 --network=deafult --ip-protocol=HTTP --ports=80 --backend-service=http-backend --backend-service-region=us-central1

              gcloud compute forwarding-rules create fr-lb-1 --region=europe-west1 --network=deafult --ip-protocol=HTTP --ports=80 --backend-service=http-backend --backend-service-region=europe-west1
    
6. Stress test an HTTP load balancer

   1. Create a VM for testing:

              gcloud compute instances create stress-test --zone us-west1-b --image mywebserver --machine-type n1-standard-1

              gcloud compute ssh stress-test

              export LB_IP=34.96.86.147

              echo $LB_IP

              ab -n 500000 -c 1000 http://$LB_IP/
