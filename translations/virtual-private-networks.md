# LAB:  Elastic Google Cloud Infrastructure: Virtual Private Networks (VPN)

## Objectives:

In this lab you learn how to perform the following tasks:
  - Create VPN gateways in each network
  - Create VPN tunnels between the gateways
  - Verify VPN connectivity

## Steps:

1. Explore the networks and instances

   1. Explore the networks:

      gcloud compute networks list

      gcloud compute networks subnets list

   2. Explore the firewall rules:

      gcloud compute firewall-rules list

   3. Explore the instances and their connectivity:

      - In the Cloud Console, on the Navigation menu (Navigation menu), click Compute Engine > VM instances:

                gcloud compute instances list

      - For server-1, click SSH to launch a terminal and connect:

                gcloud compute ssh server-1

      - To test connectivity to server-2's external IP address, run the following command, replacing server-2's     external IP address with the value noted earlier:

                ping -c 3 34.95.233.120

      - To test connectivity to server-2's internal IP address, run the following command, replacing server-2's  internal IP address with the value noted earlier:

                ping -c 3 10.128.0.2

      - Exit the SSH terminal:

                exit

      - For server-2, click SSH to launch a terminal and connect:

                gcloud compute ssh server-2

      - To test connectivity to server-1's external IP address, run the following command, replacing server-1's external IP address with the value noted earlier:

                ping -c 3 35.74.123.80

      - To test connectivity to server-1's internal IP address, run the following command, replacing server-1's internal IP address with the value noted earlier:

                ping -c 3 10.32.0.2

      - Exit the SSH terminal:
      
                exit

2. Create the VPN gateways and tunnels

   1. Reserve two static IP addresses:

      - Reserve static address: vpn1-static-ip:

                  gcloud compute addresses create vpn-1-static-ip --region us-central1 --ip-version IPV4

      - Reserve static address: vpn2-static-ip:

                  gcloud compute addresses create vpn-2-static-ip --region europe-west1 --ip-version IPV4

   2. Create the vpn-1 gateway and tunnel1to2:

      gcloud compute target-vpn-gateways create vpn-1 --region us-central1 --network vpn-network-1

      gcloud compute forwarding-rules create vpn-1-rule-esp --region us-central1 --address vpn-1-static-ip
      --ip-protocol ESP --target-vpn-gateway vpn-1 

      gcloud compute forwarding-rules create vpn-1-rule-udp500 --region us-central1 --address vpn-1-static-ip
      --ip-protocol UDP --ports 500 --target-vpn-gateway vpn-1

      gcloud compute forwarding-rules create vpn-1-rule-udp4500 --region us-central1 --address vpn-1-static-ip
      --ip-protocol UDP --ports 4500 --target-vpn-gateway vpn-1

      gcloud compute vpn-tunnels create tunnel1to2 --region us-central1 --peer-address vpn-2-static-ip --shared-secret gcprocks --ike-version 2 --local-traffic-selector 0.0.0.0/0 target-vpn-gateway vpn-1

      gcloud compute routes create tunnel1to2-route-1 --network vpn-network-1 --next-hop-vpn-tunnel tunnel1to2 --next-hop-vpn-tunnel-region us-central1 --destination-range 10.1.3.0/24

   3. Create the vpn-2 gateway and tunnel2to1:

      gcloud compute target-vpn-gateways create vpn-2 --region europe-west1 --network vpn-network-2

      gcloud compute forwarding-rules create vpn-2-rule-esp --region europe-west1 --address vpn-2-static-ip
      --ip-protocol ESP --target-vpn-gateway vpn-2

      gcloud compute forwarding-rules create vpn-2-rule-udp500 --region europe-west1 --address vpn-2-static-ip --ip-protocol UDP --ports 500 --target-vpn-gateway vpn-2

      gcloud compute forwarding-rules create vpn-2-rule-udp4500 --region europe-west1 --address vpn-2-static-ip --ip-protocol UDP --ports 4500 --target-vpn-gateway vpn-2

      gcloud compute vpn-tunnels create tunnel2to1 --region europe-west1 --peer-address vpn-1-static-ip --shared-secret gcprocks --ike-version 2 --local-traffic-selector 0.0.0.0/0 target-vpn-gateway vpn-2

      gcloud compute routes create tunnel2to1-route-1 --network vpn-network-2 --next-hop-vpn-tunnel tunnel2to1 --next-hop-vpn-tunnel-region europe-west1 --destination-range 10.5.4.0/24

3. Verify VPN connectivity

   1. Verify server-1 to server-2 connectivity:

      gcloud compute instances list

      gcloud compute ssh server-1

      ping -c 3 10.128.0.2

      exit

      gcloud compute instances list

      gcloud compute ssh server-2

      ping -c 3 10.132.0.2

   2. Remove the external IP addresses

      gcloud compute instances list

      gcloud compute instances stop server-1

      gcloud compute instances describe server-1

      gcloud compute instances delete-access-config server-1 --access-config-name external-nat

      gcloud compute instances start server-1

      gcloud compute instances describe