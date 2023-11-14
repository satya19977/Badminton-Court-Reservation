# We install SSL/TLS Certificate on our ALB so that our data in-transit is encrypted.

## Step 1. Create a public Certificate using ACM

### Step 1.1
![Screenshot (1467)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/a093b660-3b08-4408-923a-8e49144bab4d)

### Step 1.2
![Screenshot (1465)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/2f6c2fb4-79c9-4d7a-9284-63eccfb78e28)

### Step 1.3
![Screenshot (1466)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/9e94ac0d-9a76-45d1-b149-620a33345c09)

### Step 1.4
![Screenshot (1468)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/436ffaf4-0798-4c72-8b84-eba19b2cae1d)

## Step 2. Make sure to check if the CNAME record is in Route53
![Screenshot (1469)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/b5e6b954-0838-4569-93a3-e332b81f95f1)

## Step 3. Go to ALB and import the certificate

### Step 3.1
![Screenshot (1470)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/2dc6b8d3-08d3-42ea-954e-1a81de1bcc41)

### Step 3.2
![Screenshot (1471)](https://github.com/satya19977/Event-Management-System-Using-Kubernetes/assets/108000447/ea85a0a1-62e6-4c00-8dcd-0e9edd5b455b)
