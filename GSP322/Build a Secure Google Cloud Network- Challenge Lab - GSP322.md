# Build a Secure Google Cloud Network- Challenge Lab - GSP322


# Commands:

======================================================
## 1st command:
======================================================
```
gcloud config set project qwiklabs-gcp-02-0693cd452d6a
export IAP_NETWORK_TAG=permit-ssh-iap-ingress-ql-316

export INTERNAL_NETWORK_TAG=permit-ssh-internal-ingress-ql-316
export HTTP_NETWORK_TAG=permit-http-ingress-ql-316
export ZONE=us-east4-b
export SSH_IAP_Network_tag=permit-ssh-iap-ingress-ql-316
```
======================================================
## 2nd command:
======================================================
```
gcloud compute firewall-rules delete open-access --quiet
gcloud compute instances add-tags bastion --tags=$SSH_IAP_Network_tag --zone=$ZONE
gcloud compute instances start bastion --zone=$ZONE
gcloud compute firewall-rules create abhiarcade\
--action=ALLOW \
--network=acme-vpc \
--rules=tcp:22 \
--source-ranges=35.235.240.0/20 \
--target-tags=$SSH_IAP_Network_tag \
--description="Allow SSH from IAP service"
gcloud compute instances add-tags bastion --tags=$SSH_IAP_Network_tag --zone=$ZONE
gcloud compute firewall-rules create abhiarcade\
--action=ALLOW \
--network=acme-vpc \
--rules=tcp:80 \
--source-ranges=0.0.0.0/0 \
--target-tags=$HTTP_Network_Tag \
--description="abhiarcade"
gcloud compute instances add-tags juice-shop --tags=$HTTP_Network_Tag --zone=$ZONE
gcloud compute firewall-rules create abhiarcade\
--action=ALLOW \
--network=acme-vpc \
--rules=tcp:22 \
--source-ranges=192.168.10.0/24 \
--target-tags=$SSH_Internal_Network_tag \
--description="abhiarcade"
gcloud compute instances add-tags juice-shop --tags=$SSH_Internal_Network_tag --zone=$ZONE
gcloud compute ssh bastion --zone=$ZONE --quiet
```

======================================================
## 3rd command:
======================================================
```
gcloud compute ssh juice-shop --internal-ip
```

======================================================
## 4th command:
======================================================
```
curl -LO raw.githubusercontent.com/gcpsolution99/GCP-solution/main/GSP322.sh

sudo chmod +x GSP322.sh
./GSP322.sh
```
-----------------------------------------------------------------------------------------------------------------------
