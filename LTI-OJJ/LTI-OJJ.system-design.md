graph TB
    subgraph "USUARIOS"
        U1[👤 Reclutadores<br/>Web/Mobile App]
        U2[👥 Candidatos<br/>Career Portal]
        U3[🔧 Administradores<br/>Admin Panel]
        U4[🔌 Sistemas Externos<br/>APIs/Webhooks]
    end

    subgraph "CDN & EDGE"
        CDN[🌐 CloudFront/Cloudflare<br/>Static Assets + Caching]
    end

    subgraph "FRONTEND TIER"
        WEB[⚛️ Web App SPA<br/>React + TypeScript<br/>Hosted: Vercel]
        MOBILE[📱 Mobile Apps<br/>React Native<br/>iOS + Android]
        PORTAL[🌐 Career Portal<br/>Web Components<br/>Embeddable]
    end

    subgraph "API GATEWAY & SECURITY"
        WAF[🛡️ WAF + DDoS<br/>Cloudflare/AWS Shield]
        LB[⚖️ Load Balancer<br/>ALB/NLB<br/>SSL Termination]
        APIGW[🚪 API Gateway<br/>Kong/AWS API GW<br/>Auth/Rate Limit/Routing]
    end

    subgraph "APPLICATION TIER - MICROSERVICES"
        direction TB
        
        subgraph "Core Services"
            AUTH[🔐 Auth Service<br/>Node.js<br/>JWT/OAuth/SAML]
            JOB[📋 Job Service<br/>Node.js<br/>Vacantes & Pipelines]
            CAND[👥 Candidate Service<br/>Node.js<br/>Candidatos & Apps]
        end

        subgraph "Intelligence"
            AI[🤖 AI Service<br/>Python/FastAPI<br/>ML/NLP/Scoring]
        end

        subgraph "Communication"
            COMM[📧 Communication<br/>Node.js<br/>Email/SMS/Push]
            NOTIF[🔔 Notification<br/>Node.js/Socket.io<br/>Real-time Updates]
        end

        subgraph "Workflow & Integration"
            INTV[📅 Interview Service<br/>Node.js<br/>Scheduling/Eval]
            AUTO[⚙️ Automation<br/>Node.js<br/>Workflows/Rules]
            INTEG[🔗 Integration<br/>Node.js<br/>3rd Party APIs]
        end

        subgraph "Support Services"
            DOC[📄 Document Service<br/>Node.js<br/>File Management]
            ANAL[📊 Analytics Service<br/>Python<br/>Metrics/Reports]
        end
    end

    subgraph "MESSAGE BROKER"
        QUEUE[📮 RabbitMQ/SQS<br/>Async Tasks<br/>Event Bus]
    end

    subgraph "DATA TIER"
        direction TB
        
        subgraph "Primary Storage"
            PGPRI[(🐘 PostgreSQL<br/>Primary DB<br/>Multi-AZ)]
            PGREP[(🐘 PostgreSQL<br/>Read Replicas<br/>Analytics)]
        end

        subgraph "Caching & Search"
            REDIS[(🔴 Redis<br/>Cache/Session<br/>Pub-Sub)]
            ELASTIC[(🔍 Elasticsearch<br/>Full-text Search<br/>Candidates/Jobs)]
        end

        subgraph "Storage & ML"
            S3[(☁️ S3/GCS<br/>Object Storage<br/>CVs/Docs)]
            VECTOR[(🧠 Vector DB<br/>Pinecone/Weaviate<br/>Embeddings)]
        end

        subgraph "Analytics"
            DWH[(📊 BigQuery/Redshift<br/>Data Warehouse<br/>Historical Data)]
        end
    end

    subgraph "EXTERNAL SERVICES"
        direction LR
        EXT1[📧 SendGrid/SES<br/>Email Provider]
        EXT2[📱 Twilio<br/>SMS Provider]
        EXT3[📅 Google/Outlook<br/>Calendar APIs]
        EXT4[💼 LinkedIn/Indeed<br/>Job Boards]
        EXT5[🎥 Zoom/Meet<br/>Video APIs]
        EXT6[💾 HRIS Systems<br/>Workday/BambooHR]
    end

    subgraph "MONITORING & OPERATIONS"
        MON[📈 Monitoring<br/>Datadog/Prometheus<br/>Grafana]
        LOG[📝 Logging<br/>ELK Stack<br/>CloudWatch]
        TRACE[🔬 Tracing<br/>Jaeger/X-Ray<br/>Distributed Tracing]
        ALERT[🚨 Alerting<br/>PagerDuty<br/>Incident Management]
    end

    subgraph "CI/CD & INFRASTRUCTURE"
        CICD[🔄 GitHub Actions<br/>CI/CD Pipeline<br/>Automated Deploy]
        K8S[☸️ Kubernetes<br/>EKS/GKE<br/>Container Orchestration]
        IaC[🏗️ Terraform<br/>Infrastructure as Code<br/>Multi-region]
    end

    %% User connections
    U1 --> CDN
    U2 --> CDN
    U3 --> CDN
    U4 --> WAF
    
    CDN --> WEB
    CDN --> MOBILE
    CDN --> PORTAL

    %% Frontend to Gateway
    WEB --> WAF
    MOBILE --> WAF
    PORTAL --> WAF

    %% Gateway flow
    WAF --> LB
    LB --> APIGW

    %% API Gateway to Services
    APIGW --> AUTH
    APIGW --> JOB
    APIGW --> CAND
    APIGW --> AI
    APIGW --> COMM
    APIGW --> INTV
    APIGW --> AUTO
    APIGW --> INTEG
    APIGW --> DOC
    APIGW --> ANAL
    APIGW --> NOTIF

    %% Service to Service communication
    JOB --> QUEUE
    CAND --> QUEUE
    AI --> QUEUE
    COMM --> QUEUE
    INTV --> QUEUE
    AUTO --> QUEUE
    DOC --> QUEUE

    %% Services to Data Tier
    AUTH --> PGPRI
    AUTH --> REDIS
    JOB --> PGPRI
    JOB --> REDIS
    CAND --> PGPRI
    CAND --> REDIS
    CAND --> ELASTIC
    AI --> PGPRI
    AI --> VECTOR
    INTV --> PGPRI
    DOC --> PGPRI
    DOC --> S3
    ANAL --> PGREP
    ANAL --> DWH
    AUTO --> PGPRI
    INTEG --> PGPRI
    COMM --> PGPRI

    %% DB Replication
    PGPRI -.->|Replication| PGREP

    %% Real-time
    NOTIF --> REDIS

    %% External integrations
    COMM --> EXT1
    COMM --> EXT2
    INTV --> EXT3
    INTV --> EXT5
    INTEG --> EXT4
    INTEG --> EXT6

    %% Monitoring connections
    AUTH -.->|Metrics| MON
    JOB -.->|Metrics| MON
    CAND -.->|Metrics| MON
    AI -.->|Metrics| MON
    COMM -.->|Metrics| MON
    INTV -.->|Metrics| MON
    AUTH -.->|Logs| LOG
    JOB -.->|Logs| LOG
    CAND -.->|Logs| LOG
    APIGW -.->|Traces| TRACE
    MON --> ALERT

    %% Infrastructure
    K8S --> AUTH
    K8S --> JOB
    K8S --> CAND
    K8S --> AI
    K8S --> COMM
    K8S --> INTV
    K8S --> AUTO
    K8S --> INTEG
    K8S --> DOC
    K8S --> ANAL
    K8S --> NOTIF
    
    CICD -.->|Deploy| K8S
    IaC -.->|Provision| K8S
    IaC -.->|Provision| PGPRI
    IaC -.->|Provision| REDIS

    %% Styling
    classDef frontend fill:#e3f2fd,stroke:#1976d2,stroke-width:2px
    classDef gateway fill:#fff3e0,stroke:#f57c00,stroke-width:2px
    classDef service fill:#e8f5e9,stroke:#388e3c,stroke-width:2px
    classDef intelligence fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    classDef data fill:#fce4ec,stroke:#c2185b,stroke-width:2px
    classDef external fill:#e0f2f1,stroke:#00796b,stroke-width:2px
    classDef ops fill:#fff9c4,stroke:#f57f17,stroke-width:2px

    class WEB,MOBILE,PORTAL frontend
    class WAF,LB,APIGW gateway
    class AUTH,JOB,CAND,COMM,INTV,AUTO,INTEG,DOC,ANAL,NOTIF service
    class AI intelligence
    class PGPRI,PGREP,REDIS,ELASTIC,S3,VECTOR,DWH,QUEUE data
    class EXT1,EXT2,EXT3,EXT4,EXT5,EXT6 external
    class MON,LOG,TRACE,ALERT,CICD,K8S,IaC ops