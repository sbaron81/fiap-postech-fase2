## Vídeo de Demonstração (até 20 min):
• Mostre o docker compose up funcionando, com todos os 9 contêineres (5 apps + 4 DBs) rodando localmente.
```bash
cd /home/sbaron/github/fiap-postech-fase2
docker compose up -d --build
docker compose ps

curl -k http://localhost:8001/health
curl -k http://localhost:8002/health
curl -k http://localhost:8003/health
curl -k http://localhost:8004/health
curl -k http://localhost:8005/health
```

• Mostre seu cluster Kubernetes provisionado na nuvem (EKS, GKE, ou AKS).
```bash
eksctl get cluster
kubectl get nodes
kubectl get deployments -n A
```

• Mostre os 5 microsserviços rodando como Pods (kubectl get pods).
```bash
kubectl get pods -n analytics-service
kubectl get pods -n auth-service
kubectl get pods -n evaluation-service
kubectl get pods -n flag-service
kubectl get pods -n targeting-service
```

• Mostre o Nginx Ingress funcionando (faça uma chamada curl ou Postman para a URL do Load Balancer criado).
```bash
aws elbv2 describe-load-balancers

curl -k https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/auth/health
curl -k https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/flags/health
curl -k https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/targeting/health
curl -k https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/evaluate/health
curl -k https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/analytics/health
```


## Demonstre a escalabilidade:
• Gere carga no evaluation-service (com hey, ab ou Postman) e mostre o HPA aumentando as réplicas (kubectl get hpa e kubectl get pods).
```bash
watch kubectl -n evaluation-service get scaledobject,hpa,pods
aws dynamodb scan --table-name ToggleMasterAnalytics --output json
aws dynamodb scan --table-name ToggleMasterAnalytics --output json | jq '.Items[-3:]'

URL="https://k8s-ingressn-ingressn-1d7f95148f-ec0fcd2c2e76dd83.elb.us-east-1.amazonaws.com/evaluate"

curl -k "${URL}/evaluate?user_id=user-001&flag_name=enable-new-dashboard"

hey -z 30s -c 50 -m GET  "${URL}/evaluate?user_id=user-002&flag_name=enable-new-dashboard"

```


• Envie manualmente várias mensagens para a fila SQS.
```bash
watch kubectl -n analytics-service get scaledobject,hpa,pods
aws dynamodb scan --table-name ToggleMasterAnalytics --output json

QUEUE_URL="https://sqs.us-east-1.amazonaws.com/343770067934/fiap-fase2"
aws sqs send-message     --queue-url "$QUEUE_URL"     --message-body "{\"user_id\":\"user-teste1\",
\"flag_name\":\"enable-new-dashboard\",\"result\":true,\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%S.000Z)\"}" 

aws sqs send-message     --queue-url "$QUEUE_URL"     --message-body "{\"user_id\":\"user-teste2\",\"flag_name\":\"enable-new-dashboard\",\"result\":true,\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%S.000Z)\"}" 

aws sqs send-message     --queue-url "$QUEUE_URL"     --message-body "{\"user_id\":\"user-teste3\",\"flag_name\":\"enable-new-dashboard\",\"result\":true,\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%S.000Z)\"}" 

for i in $(seq 1 20); do   
   aws sqs send-message     --queue-url "$QUEUE_URL"     --message-body "{\"user_id\":\"user-$i\",\"flag_name\":\"enable-new-dashboard\",\"result\":true,\"timestamp\":\"$(date -u +%Y-%m-%dT%H:%M:%S.000Z)\"}"     > /dev/null & 
done

```

• Mostre o HPA (ou KEDA, se usou) do analytics-service detectando a carga (CPU ou fila) e escalando seus pods.
```bash
watch kubectl -n analytics-service get scaledobject,hpa,pods
watch kubectl -n evaluation-service get scaledobject,hpa,pods
```

• Mostre os dados aparecendo na tabela do DynamoDB.
```bash
aws dynamodb scan --table-name ToggleMasterAnalytics --output json
aws dynamodb scan --table-name ToggleMasterAnalytics --output json | jq '.Items[-3:]'
```

• Faça uma breve explicação da arquitetura e dos desafios encontrados (ex: limitações da LabRole).

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


• Explique como você implementou a escalabilidade do analyticsservice (HPA por CPU ou KEDA por fila) e justifique a escolha.

 * Utilizei o KEDA para monitar as filas SQS e fazer a escalabilidade conforme a quantidade de mensagens aumente na fila
  

• Explique a diferença de propósito entre os 3 data stores utilizados (RDS, ElastiCache e DynamoDB).

  * RDS:  Gravar dados permanente de forma estruturada
  * ElastiCache: Mantem um cache das consultas em memória, muito rápido. 
  * DynamoDB: Gravar dados não-estruturado em grande volume. Muito rápido também.
  

## Relatório de Entrega (.PDF ou .txt):
• Nomes dos participantes, RM e usernames do Discord.
Nome: Sidnei Baron
RM: rm370250
Discord: sbaron81

• Link dos repositórios.
https://github.com/sbaron81/fiap-postech-fase2

• Link do vídeo salvo no Youtube ou lugar de sua preferência.
https://youtu.be/YyUNaFNj0Ms
