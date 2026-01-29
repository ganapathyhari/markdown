# markdown
flowchart LR
    %% ==== Styles ====
    classDef lb fill:#E3F2FD,stroke:#1565C0,stroke-width:2px;
    classDef db2 fill:#FFF3E0,stroke:#EF6C00,stroke-width:2px;
    classDef svc fill:#E8F5E9,stroke:#2E7D32,stroke-width:2px;
    classDef storage fill:#F3E5F5,stroke:#6A1B9A,stroke-width:2px;
    classDef dns fill:#FFEBEE,stroke:#C62828,stroke-width:2px;

    %% ==== Components ====
    subgraph BAW_Runtime ["Business Automation Workflow (BAW)"]
        BAW[BAW Application Server]:::svc
        Cred[Credential Store (IAM token / API key)]:::svc
    end

    subgraph IBM_Cloud_VPC ["IBM Cloud VPC"]
        DNS[IBM Cloud DNS (CNAME → LB)]:::dns
        LB[IBM Cloud Load Balancer]:::lb
        DB2_Primary[DB2 SaaS – Primary Region]:::db2
        DB2_Replica[DB2 SaaS – Secondary Region]:::db2
        KMS[IBM Key Protect / External KMIP/HSM]:::svc
        Backup[Encrypted Backup Store (Object Storage)]:::storage
    end

    %% ==== Connections ====
    BAW -- JDBC (sslConnection=true, auth=IAM) --> LB
    Cred --> BAW
    LB -->|Primary IP (A‑record)| DB2_Primary
    LB -->|Fail‑over IP (B‑record)| DB2_Replica
    DNS --> LB
    DB2_Primary --> KMS
    DB2_Replica --> KMS
    DB2_Primary --> Backup
    DB2_Replica --> Backup

    %% ==== Optional notes ====
    class BAW,Cred,DB2_Primary,DB2_Replica,KMS,Backup,LB,DNS svc;
