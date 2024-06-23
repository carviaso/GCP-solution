Con google cloud 
1.	Check the firewall rules. Remove the overly permissive rules
2.	Navigate to Compute Engine in the Cloud console and identify the bastion host. The instance should be stopped. Start the instance permit-ssh-iap-ingress-ql-316.
3.	The bastion host is the one machine authorized to receive external SSH traffic. Create a firewall rule that allows SSH (tcp/22) from the IAP service. The firewall rule must be enabled for the bastion host instance using a network tag of permit-http-ingress-ql-316.
4.	The juice-shop server serves HTTP traffic. Create a firewall rule that allows traffic on HTTP (tcp/80) to any address. The firewall rule must be enabled for the juice-shop instance using a network tag of permit-http-ingress-ql-316.
5.	You need to connect to juice-shop from the bastion using SSH. Create a firewall rule that allows traffic on SSH (tcp/22) from acme-mgmt-subnet network address. The firewall rule must be enabled for the juice-shop instance using a network tag of permit-ssh-internal-ingress-ql-316.
6.	In the Compute Engine instances page, click the SSH button for the bastion host. Once connected, SSH to juice-shop.


Aquí tienes un script detallado en bash para realizar cada uno de los pasos descritos usando gcloud:

```bash

#!/bin/bash

# Variables
BASTION_INSTANCE="bastion-host-instance"
JUICE_SHOP_INSTANCE="juice-shop-instance"
BASTION_TAG="permit-ssh-iap-ingress-ql-316"
JUICE_SHOP_HTTP_TAG="permit-http-ingress-ql-316"
JUICE_SHOP_SSH_TAG="permit-ssh-internal-ingress-ql-316"
REGION="us-west1"
ZONE="us-west1-c"
ACME_MGMT_SUBNET="10.0.0.0/24"  # Reemplaza con el rango de IP de tu subred acme-mgmt-subnet

# Función para verificar el estado de la ejecución de una tarea
check_status() {
    if [ $? -eq 0 ]; then
        echo "✅ Tarea completada exitosamente."
    else
        echo "❌ Error: La tarea ha fallado. Saliendo del script."
        exit 1
    fi
}

# 1. Check the firewall rules and remove the overly permissive rules
echo "Verificando y eliminando reglas de firewall demasiado permisivas..."
# Ejemplo: Asegúrate de especificar las reglas de firewall que deben ser eliminadas
gcloud compute firewall-rules list --filter="name~'overly-permissive'" --format="value(name)" | while read -r RULE_NAME; do
  gcloud compute firewall-rules delete $RULE_NAME -q
  check_status
done

# 2. Start the bastion host instance
echo "Iniciando la instancia bastion host..."
gcloud compute instances start $BASTION_INSTANCE --zone=$ZONE
check_status

# 3. Create a firewall rule that allows SSH (tcp/22) from the IAP service for the bastion host
echo "Creando la regla de firewall para permitir SSH desde el servicio IAP para bastion host..."
gcloud compute firewall-rules create allow-ssh-from-iap \
    --action=ALLOW \
    --rules=tcp:22 \
    --source-ranges=35.235.240.0/20 \
    --target-tags=$BASTION_TAG \
    --description="Allow SSH from IAP service to bastion host"
check_status

# 4. Create a firewall rule that allows HTTP traffic (tcp/80) to any address for the juice-shop instance
echo "Creando la regla de firewall para permitir tráfico HTTP a cualquier dirección para juice-shop..."
gcloud compute firewall-rules create allow-http-to-juice-shop \
    --action=ALLOW \
    --rules=tcp:80 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=$JUICE_SHOP_HTTP_TAG \
    --description="Allow HTTP traffic to juice-shop instance"
check_status

# 5. Create a firewall rule that allows SSH (tcp/22) from acme-mgmt-subnet for the juice-shop instance
echo "Creando la regla de firewall para permitir SSH desde acme-mgmt-subnet para juice-shop..."
gcloud compute firewall-rules create allow-ssh-from-acme-mgmt-subnet \
    --action=ALLOW \
    --rules=tcp:22 \
    --source-ranges=$ACME_MGMT_SUBNET \
    --target-tags=$JUICE_SHOP_SSH_TAG \
    --description="Allow SSH traffic from acme-mgmt-subnet to juice-shop instance"
check_status

# Agregar las etiquetas de red a las instancias correspondientes
echo "Agregando etiqueta de red a la instancia bastion..."
gcloud compute instances add-tags $BASTION_INSTANCE --zone=$ZONE --tags=$BASTION_TAG
check_status

echo "Agregando etiquetas de red a la instancia juice-shop..."
gcloud compute instances add-tags $JUICE_SHOP_INSTANCE --zone=$ZONE --tags=$JUICE_SHOP_HTTP_TAG,$JUICE_SHOP_SSH_TAG
check_status

# 6. SSH to the bastion host via IAP
echo "Conectándose a la instancia bastion host a través de IAP..."
gcloud compute ssh $BASTION_INSTANCE --zone=$ZONE --tunnel-through-iap --command="
  echo 'Conectado a bastion host. Ahora conectándose a juice-shop...'
  ssh -i ~/.ssh/id_rsa user@juice-shop-internal-ip
"
check_status
```

Explicación del Script:

    Eliminar Reglas de Firewall Demasiado Permisivas: Este paso lista las reglas de firewall con nombres que contienen "overly-permissive" y las elimina.
    Iniciar la Instancia Bastion Host: Este comando inicia la instancia bastion host especificada.
    Crear Regla de Firewall para SSH desde IAP: Esta regla permite el tráfico SSH (tcp/22) desde el servicio IAP a la instancia bastion host.
    Crear Regla de Firewall para Tráfico HTTP: Esta regla permite el tráfico HTTP (tcp/80) a la instancia juice-shop desde cualquier dirección.
    Crear Regla de Firewall para SSH desde Acme-Mgmt-Subnet: Esta regla permite el tráfico SSH (tcp/22) desde la subred acme-mgmt-subnet a la instancia juice-shop.
    Agregar Etiquetas de Red a las Instancias: Este paso agrega las etiquetas de red necesarias a las instancias bastion y juice-shop.
    Conectarse a la Instancia Bastion Host a través de IAP: Este comando se conecta a la instancia bastion host usando IAP y luego establece una conexión SSH a la instancia juice-shop.

Notas Adicionales:

    Reemplaza las Variables: Asegúrate de reemplazar las variables como BASTION_INSTANCE, JUICE_SHOP_INSTANCE, ACME_MGMT_SUBNET y cualquier otra según tu configuración específica.
    Permisos Necesarios: Asegúrate de tener los permisos necesarios para realizar estas operaciones en tu proyecto de Google Cloud.
    Claves SSH: Asegúrate de que las claves SSH estén configuradas correctamente para acceder a las instancias.

Este script proporciona un flujo automatizado para configurar y gestionar las reglas de firewall y las conexiones SSH en Google Cloud Platform, con validaciones para garantizar que cada paso se complete correctamente.
