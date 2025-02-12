```mermaid
   sequenceDiagram
    participant User
    participant EUDI Wallet
    participant Service Provider
    box CSC Protocol Usage
    participant Signing Service 
    participant RQES Provider
    end

    Service Provider-->Service Provider: document signing prepare
    Service Provider-->Signing Service: signature request
    
    Note over Signing Service, RQES Provider: Phase 1: Certificate Listing and Selection
    
    alt via authlogin
    Signing Service->>User: ask for authentication
    User->>Signing Service: User Login to Signing Service (eg. via Username, Password, VCs)
    Signing Service->>RQES Provider: user authentication 
    else via oauth2/authorize (scope service)
    Signing Service->>RQES Provider: oauth/authorize
    User->>RQES Provider: User Login to RQES Provider directly
    end

    RQES Provider->>Signing Service: bearer token provisioning

    Signing Service->>RQES Provider: POST /csc/v2/credentials/list
    RQES Provider->>Signing Service: { credentialIDs: [...], credentialInfos: [...] }
    opt more than one enabled certificate 
    User->>Signing Service: credential selection
    end

    Note over User, RQES Provider: Phase 2: Signature Confirmation & Private Key Unlocking (Credential Authorization)
    
    Signing Service->>RQES Provider: Request Signing of Document: oauth2/authorize (hashes, URIs, cert identifier)
    RQES Provider->>User: PID, signature transaction authorization Presentation
    EUDI Wallet->>RQES Provider: PID Presentation, transaction selfsigned authorization via OID4VP
    
    RQES Provider->>Signing Service: SAD
    
    Note over Signing Service, RQES Provider: Phase 3: Signature Creation
    Signing Service->>RQES Provider: POST /csc/v2/signatures/signHash
    RQES Provider->>Signing Service: Signed Hash

    Note over User, Signing Service: Phase 6: Signed Document Formation and Retrieval
    Signing Service->>Service Provider: Signed Document
    Service Provider->>User: Signed Document
```
