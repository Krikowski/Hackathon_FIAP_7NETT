# Hackathon_FIAP_7NETT

## Introdução

Este projeto foi desenvolvido para o Hackathon FIAP 7NETT, atendendo ao desafio da Visionary Analytics: criar uma solução escalável e robusta para upload, processamento e análise de vídeos, com foco na detecção de QR Codes.
A solução é composta por dois microsserviços independentes, mas interconectados via RabbitMQ:

🪺 VideoNest → responsável por upload, armazenamento inicial e enfileiramento de vídeos.
⚒️ ScanForge → responsável por processamento pesado, extração de frames e detecção de QR Codes.

Essa divisão respeita práticas de arquitetura orientada a serviços (SOA) e garante desacoplamento, resiliência e escalabilidade horizontal.

# 🪺 VideoNest – O Ninho de Vídeos
Link para repositório: https://github.com/Krikowski/VideoNest/blob/main/README.md

O VideoNest é o microsserviço que recebe os vídeos brutos dos usuários e os organiza em um “ninho seguro” antes de enviá-los para análise.

- Funcionalidades (RF1, RF2, RF6, RF7):
- Upload de vídeos (.mp4, .avi, .mov, .mkv até 100MB).
- Validação de formatos e tamanhos.
- Persistência inicial no MongoDB.
- Publicação assíncrona de mensagens no RabbitMQ.
- Consulta de status e resultados de vídeos.

Bônus Implementados:
- Redis para cache de status.
- Swagger para documentação.
- Logs estruturados com Serilog.

# ⚒️ ScanForge – A Forja de Escaneamento
Link para repositório: https://github.com/Krikowski/ScanForge/blob/main/README.md

O ScanForge é o motor de processamento da arquitetura: recebe os vídeos via fila RabbitMQ e aplica o “fogo da forja” para extrair informações valiosas.

- Funcionalidades (RF3, RF4, RF5):
- Consumo de mensagens do RabbitMQ.
- Extração de frames com FFMpegCore.
- Detecção e decodificação de QR Codes (ZXing.NET).
- Atualização de status do vídeo.
- Armazenamento dos resultados (conteúdo dos QR Codes + timestamps) no MongoDB.

Bônus Implementados:
- Notificações em tempo real com SignalR.
- Monitoramento com Prometheus.
- Retries automáticos e DLQ no RabbitMQ.

### Arquitetura Geral
 [ Cliente / Usuário ]
          |
   (upload via API)
          ↓
     🪺 VideoNest
          |
   (mensagem na fila)
          ↓
     RabbitMQ Broker
          |
          ↓
     ⚒️ ScanForge
          |
   (processamento pesado)
          ↓
     MongoDB + Redis
          |
   (status, resultados, QR Codes)
          ↓
 [ Cliente consulta resultados ]

➡️ Essa arquitetura garante desacoplamento: se o processamento falhar, o upload continua funcionando.
➡️ Inspirada em players reais como YouTube, Netflix e e-commerces de alta escala.

### Justificativas Técnicas

.NET 8.0 → performance, tipagem forte, integração nativa com bibliotecas de vídeo e mensageria.
RabbitMQ → filas assíncronas, DLQ, simplicidade para workloads de upload.
MongoDB → flexibilidade para armazenar vídeos e resultados de análise.
Redis → respostas rápidas para consultas de status.
SignalR → experiência em tempo real para usuários.
Prometheus → monitoramento de métricas em produção.
Docker → containerização multiplataforma e preparação para Kubernetes.
Clean Code, SOLID, DDD, KISS, YAGNI → práticas que reduzem débito técnico e aumentam manutenibilidade.

### Relatório de Testes Consolidado
#### 🪺 VideoNest
Cobertura: Upload, validação de formatos/tamanho, DTOs, repositório MongoDB, RabbitMQ Publisher, APIs de consulta.
Impacto: garante confiabilidade no upload e resiliência em falhas de mensageria.
Quantidade: ~15–18 testes unitários.
Valor: reduz risco de uploads inválidos, garante UX estável, protege integrações.

#### ⚒️ ScanForge
Cobertura: Controllers (health check), DTOs, models (VideoResult/QRCodeResult), repositório MongoDB, SignalRNotifierService, VideoProcessingService.
Impacto: protege contra falhas críticas em processamento pesado e garante logs/rastreabilidade.
Quantidade: ~25 testes unitários.
Valor: alta confiabilidade, simula cenários reais de falha, prepara o sistema para escala.

### Consolidação

Total de testes: ~40–43 unitários.
Abrangência: cobre desde upload até processamento final, incluindo falhas, retries, cache, logs e integrações externas.
Resultado: uma suíte de testes que funciona como documentação executável da solução, aumentando a confiança para produção.

Instalação e Uso

Pré-requisitos
Docker
.NET SDK 8.0
RabbitMQ
MongoDB
Redis

Execução
Subir com Docker Compose
docker-compose up --build

Endpoints

VideoNest
POST /api/videos → upload de vídeo
GET /api/videos/{id}/status → status do processamento
GET /api/videos/{id}/resultado → resultado do processamento


ScanForge

GET /health → status do serviço
GET /results/{id} → resultados do processamento

Documentação
Swagger disponível em /swagger.
