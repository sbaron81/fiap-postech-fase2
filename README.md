# fiap-postech-fase2

## Problemas encontrados

* auth-service
  * Atualizado lib pgx para v5
```bash
 > [builder 4/5] RUN go mod tidy:
0.204 go: errors parsing go.mod:
0.204 go.mod:18:2: require github.com/jackc/pgx/v4/stdlib: version "v4.18.3" invalid: should be v0 or v1, not v4
```

  * Removido lib não utilizadas
```bash
./handlers.go:4:2: "crypto/sha256" imported and not used
./handlers.go:5:2: "encoding/hex" imported and not used
./key.go:7:2: "fmt" imported and not used
./main.go:5:2: "fmt" imported and not used
```

* flag-service
  * Fixado versão do Werkzeug
```bash 
Werkzeug==2.2.2
```



## AWS

### EKS:
beautiful-unicorn-1783889869
https://CDA1416759FEB693398C42F12427CB4A.gr7.us-east-1.eks.amazonaws.com


### ECR:
343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/analytics-service
343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/auth-service
343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/evaluation-service
343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/flag-service
343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/targeting-service

aws ecr get-login-password | docker login --username AWS --password-stdin 343770067934.dkr.ecr.us-east-1.amazonaws.com


docker tag fiap-postech-fase2-auth_service:latest 343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/auth-service:latest
docker push 343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/auth-service:latest


docker tag fiap-postech-fase2-analytics_service:latest 343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/analytics-service:latest
docker push 343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/analytics-service


docker tag fiap-postech-fase2-evaluation_service:latest 343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/evaluation-service:latest
docker push 343770067934.dkr.ecr.us-east-1
docker push 343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/evaluation-service


docker tag fiap-postech-fase2-flag_service:latest 343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/flag-service:latest
docker push 343770067934.dkr.ecr.us-east-1
docker push 343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/flag-service


docker tag fiap-postech-fase2-targeting_service:latest 343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/targeting-service:latest
docker push 343770067934.dkr.ecr.us-east-1
docker push 343770067934.dkr.ecr.us-east-1.amazonaws.com/fiap/targeting-service



### RDS: 

Username: postgres
Pass: Mypasswordaws.123
Pass64: TXlwYXNzd29yZGF3cy4xMjM=

auth-service.c3ivnpwh3dp3.us-east-1.rds.amazonaws.com
flag-service.c3ivnpwh3dp3.us-east-1.rds.amazonaws.com
targeting-service.c3ivnpwh3dp3.us-east-1.rds.amazonaws.com

echo -n 'postgres://postgres:Mypasswordaws.123@auth-service.c3ivnpwh3dp3.us-east-1.rds.amazonaws.com:5432/postgres' | base64
cG9zdGdyZXM6Ly9wb3N0Z3JlczpNeXBhc3N3b3JkYXdzLjEyM0BhdXRoLXNlcnZpY2UuYzNpdm5wd2gzZHAzLnVzLWVhc3QtMS5yZHMuYW1hem9uYXdzLmNvbTo1NDMyL3Bvc3RncmVz

echo -n 'postgres://postgres:Mypasswordaws.123@flag-service.c3ivnpwh3dp3.us-east-1.rds.amazonaws.com:5432/postgres' | base64
cG9zdGdyZXM6Ly9wb3N0Z3JlczpNeXBhc3N3b3JkYXdzLjEyM0BmbGFnLXNlcnZpY2UuYzNpdm5wd2gzZHAzLnVzLWVhc3QtMS5yZHMuYW1hem9uYXdzLmNvbTo1NDMyL3Bvc3RncmVz

echo -n 'postgres://postgres:Mypasswordaws.123@targeting-service.c3ivnpwh3dp3.us-east-1.rds.amazonaws.com:5432/postgres' | base64
cG9zdGdyZXM6Ly9wb3N0Z3JlczpNeXBhc3N3b3JkYXdzLjEyM0B0YXJnZXRpbmctc2VydmljZS5jM2l2bnB3aDNkcDMudXMtZWFzdC0xLnJkcy5hbWF6b25hd3MuY29tOjU0MzIvcG9zdGdyZXM=

kubectl run pg-test --rm -i --tty --image=postgres:17-alpine -- sh


### ElasticCache

evaluation-service-t0uqzp.serverless.use1.cache.amazonaws.com:6379

### DynamoDB:

AWS_DYNAMODB_TABLE: ToggleMasterAnalytics
AWS_REGION: us-east-1


### SQS: 
https://sqs.us-east-1.amazonaws.com/343770067934/fiap-fase2


### Kubernetes deploy

aws eks update-kubeconfig --region us-east-1 --name beautiful-unicorn-1783889869

kubectl apply -f https://github.com/kubernetes-sigs/metricsserver/releases/latest/download/components.yaml

eksctl utils associate-iam-oidc-provider --cluster=beautiful-unicorn-1783889869 --region=us-east-1 --approve

eksctl create iamserviceaccount \
  --name nginx-ingress-serviceaccount \
  --namespace ingress-nginx \
  --cluster beautiful-unicorn-1783889869 \
  --region us-east-1 \
  --attach-policy-arn arn:aws:iam::aws:policy/ElasticLoadBalancingFullAccess \
  --approve \
  --override-existing-serviceaccounts


helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --values ingress-nginx/values.yaml


  ### Deploy 

kubectl apply -f eks/00-namespaces.yaml

kubectl apply -R -f eks/auth-service
kubectl apply -R -f eks/analytics-service
kubectl apply -R -f eks/evaluation-service
kubectl apply -R -f eks/flag-service
kubectl apply -R -f eks/targeting-service

kubectl apply -f eks/ingress.yaml



curl -k https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/auth/health
{"status":"ok"}

curl -k https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/flags/health
{"status":"ok"}

curl -k https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/targeting/health
{"status":"ok"}

curl -k https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/evaluate/health
{"status":"ok"}

curl -k https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/analytics/health
{"status":"ok"}



### Auth-service
```bash
URL="https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/auth"

curl -k -X POST ${URL}/admin/keys \
-H "Content-Type: application/json" \
-H "Authorization: Bearer admin-secreto-123" \
-d '{"name": "meu-primeiro-servico"}'
```
```json
{"name":"meu-primeiro-servico","key":"tm_key_2c9f1040ffe6b49adf062c741ea531ed6729727b36ff1e159fe0f5e4ce3d551b","message":"Guarde esta chave com segurança! Você não poderá vê-la novamente."}
```

```bash
curl -k ${URL}/validate \
-H "Authorization: Bearer tm_key_2c9f1040ffe6b49adf062c741ea531ed6729727b36ff1e159fe0f5e4ce3d551b"
```
```json
{"message":"Chave válida"}
```

```bash
URL="https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/auth"

curl -k -X POST ${URL}/admin/keys \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer admin-secreto-123" \
    -d '{"name": "evaluation-service-key"}'

{"name":"evaluation-service-key","key":"tm_key_f3df7e875c62635cda00019eb4ae9b4c700677eff5666cb6850fa4d64fbedee6","message":"Guarde esta chave com segurança! Você não poderá vê-la novamente."}

```    

### Flag-service

```bash
URL="https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/flags"
CHAVE="tm_key_2c9f1040ffe6b49adf062c741ea531ed6729727b36ff1e159fe0f5e4ce3d551b"

curl -k -X POST ${URL}/flags \
-H "Content-Type: application/json" \
-H "Authorization: Bearer ${CHAVE}" \
-d '{
    "name": "enable-new-dashboard",
    "description": "Ativa o novo dashboard para usuários",
    "is_enabled": true
}'
```

```bash
curl -k ${URL}/flags \
-H "Authorization: Bearer ${CHAVE}"
```
```json
[{"created_at":"Mon, 13 Jul 2026 01:33:06 GMT","description":"Ativa o novo dashboard para usu\u00e1rios","id":1,"is_enabled":true,"name":"enable-new-dashboard","updated_at":"Mon, 13 Jul 2026 01:33:06 GMT"}]
```

```bash
curl -k -X PUT ${URL}/flags/enable-new-dashboard \
-H "Content-Type: application/json" \
-H "Authorization: Bearer ${CHAVE}" \
-d '{"is_enabled": false}'
```

### Targeting-service
```bash
URL="https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/targeting"
CHAVE="tm_key_2c9f1040ffe6b49adf062c741ea531ed6729727b36ff1e159fe0f5e4ce3d551b"

curl -k -X POST ${URL}/rules \
-H "Content-Type: application/json" \
-H "Authorization: Bearer ${CHAVE}" \
-d '{
    "flag_name": "enable-new-dashboard",
    "is_enabled": true,
    "rules": {
        "type": "PERCENTAGE",
        "value": 50
    }
}'


curl -k ${URL}//rules/enable-new-dashboard \
-H "Authorization: Bearer ${CHAVE}"

curl -k -X PUT ${URL}/rules/enable-new-dashboard \
-H "Content-Type: application/json" \
-H "Authorization: Bearer ${CHAVE}" \
-d '{
    "rules": {
        "type": "PERCENTAGE",
        "value": 75
    }
}'
```

### Evaluation-service
```bash
URL="https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/evaluate"
CHAVE="tm_key_2c9f1040ffe6b49adf062c741ea531ed6729727b36ff1e159fe0f5e4ce3d551b"

CHAVE2="tm_key_f3df7e875c62635cda00019eb4ae9b4c700677eff5666cb6850fa4d64fbedee6"

curl -k "${URL}/evaluate?user_id=user-123&flag_name=enable-new-dashboard"

curl -k "${URL}/evaluate?user_id=user-abc&flag_name=enable-new-dashboard"

```

### Analytics-service
```bash
URL="https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/evaluate"

curl -k "${URL}/evaluate?user_id=test-user-1&flag_name=enable-new-dashboard"
curl -k "${URL}/evaluate?user_id=test-user-2&flag_name=enable-new-dashboard"
```


### hey
```bash
URL="https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/evaluate"

hey -z 30s -c 50 -m GET  "${URL}/evaluate?user_id=user-123&flag_name=enable-new-dashboard"
```

kubectl apply -f eks/analytics-service/scaledobject.yaml
watch kubectl -n analytics-service get scaledobject,hpa,pods

```bash
QUEUE_URL="https://sqs.us-east-1.amazonaws.com/343770067934/fiap-fase2"


aws sqs send-message \
  --queue-url "$QUEUE_URL" \
  --message-body "{\"user_id\":\"user-321\",\"flag_name\":\"enable-new-dashboard\",\"result\":true,\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%S.000Z)\"}"


for i in $(seq 1 20); do
  aws sqs send-message \
    --queue-url "$QUEUE_URL" \
    --message-body "{\"user_id\":\"user-$i\",\"flag_name\":\"enable-new-dashboard\",\"result\":true,\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%S.000Z)\"}" \
    > /dev/null &
done
wait
echo "20 mensagens enviadas"
```