# BU1 KYS MVP Workflow

MVP Restrictions:
## Company Perspective
- Any person can trigger the process of supplier onboarding
- The company is authorized to present attestations and receive attestations (no configuration support)
- Mutual authentication is set to default true (no TLOL or device-binding checks are applied).
- The MVP process is executed sequentially in one step. 

## RelyingParty perspective
- The company who wants to be supplier will be classified as low/medium risk supplier. It will be no high-risk supplier  (therefore, e.g.: no sanction screening is required)

MVP+ Extension:
## RelyingParty perspective
- Additionally support for the KYS due diligence
- Additionally support for the sanction list validation 
- Additionally support for the ESG Cetificates 

## Pre-requisites
This are the pre-requisites for the company in order to run the MVP.


```mermaid
sequenceDiagram
    participant Auth.Source 
    note right of Auth.Source : Bundesanzeiger, KVK, ...
    participant TAX_Administration 
    participant Company
    participant RelyingParty
    participant Bank
    
    Auth.Source ->> Company : issue EBWOID  
    Auth.Source ->> Company : issue EUCC
    Auth.Source ->> RelyingParty : issue EBWOID
    
    alt PubEAA Issuer available
        TAX_Administration ->> Company : issue TAX
    else EAA attestation issuing
        Company ->> Company: issue TAX
    end
    
    Company ->> Company: issue VAT, CompanyInfo,  ContactPerson
    Company ->> Company: issue PaymentTerms, SiteAttesttion 
    Bank ->> Company: issue IBAN 
    
    participant LEI      
    LEI ->> Company: issue LEI 
    participant GS1
    GS1 ->> Company: issue GS1 
    
    Note over Source, Company_Wallet: part of the MVP++
    Company ->> Company: issue OwnershipList,ControlList, UBO
    Company ->> Company: issue DUNS, ESG 
    Company ->> Company: issue TFS       
```

### 1. Scenario KYC 

### 1.1. Legal Entity Selection
```mermaid
sequenceDiagram
    actor Initiator
    activate Initiator
    Person->>+RP_Portal: Select "start supplier onboarding" Service
    alt Wallet_Support_EndPoint (ex. EUBW DirectoryList)
        Bank_Portal->>+Bank_Portal : Provide the list of available legal entities
        Initiator->>+Bank_Portal: select the legal entity from the list & the respective wallet address
        Bank_Portal->>+Bank_Portal: resolve the endpoint of selected legal entity
    else Wallet_Support_EndPoint (ex. Resolvable eAddress or public endpoint URI)
        RP_Portal->>+RP_Portal : Provide an input field
        Initiator->>+RP_Portal: fill the address or end-point of the business wallet
        RP_Portal->>+RP_Portal: resolve eAddress
    else Support directly into EUBW  
        Note over Company_Wallet: the company wallet already integrate the business process of specific banks
        Initiator->>+Company_Wallet: Select bank in the EUBW (configured in wallet)
    else Other: manual process (EUBW or EUDI Wallet)
        Note over Company_Wallet: manuall proces by the Initiator
    end
    Person->>+RP_Portal: trigger process
```


### 1.2. LegalEntity Identification 

```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal<<->>Bank_Wallet: generate proof-request
    RP_Portal<<->>Bank_Wallet: for EBWOID, EUCC,TAX, VAT
    alt Automatically (EUBW support end-points)
        RP_Portal->>+Company_Wallet: request presentations 
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI for the request
        Initiator->>+Company_Wallet: copy/paste openid4vp-URI into the company wallet or scan the QRCode
    end
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own business configuration)
    Company_Wallet->>RP_Portal: present the attestations
    RP_Portal<<->>Bank_Wallet: verification of attestations rulebooks
```

### 1.3. KYS - Base Information 
```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal<<->>Bank_Wallet: generate proof-request
    RP_Portal<<->>Bank_Wallet: for CompanyInfo, ContactPerson, PaymentTerms
    alt Automatically (EUBW support end-points)
        RP_Portal->>+Company_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Company_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Company_Wallet->>RP_Portal: present the attestations
    RP_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
```

### 1.4. KYS - Payment Information
```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal<<->>Bank_Wallet: generate proof-request
    RP_Portal<<->>Bank_Wallet: for IBAN  
    alt Automatically (EUBW support end-points)
        RP_Portal->>+Company_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Company_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Company_Wallet->>RP_Portal: present the attestations
    RP_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
```

### 1.5. KYS - Additionally identifier Information  
```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal<<->>Bank_Wallet: generate proof-request
    RP_Portal<<->>Bank_Wallet: for DUNS, GS1, LEI 
    RP_Portal<<->>Bank_Wallet: in case that the company has site informaiton : Site 
    alt Automatically (EUBW support end-points)
        RP_Portal->>+Company_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Company_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Company_Wallet->>RP_Portal: present the attestations
    RP_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
```

### 1.6. KYS - CDD Information  (this will be handled in the MVP+)
```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal<<->>Bank_Wallet: generate proof-request
    RP_Portal<<->>Bank_Wallet: for OwnershipList,ControlList -> only the relevant KYS attributes ( GDPR conform)  
    alt Automatically (EUBW support end-points)
        RP_Portal->>+Company_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Company_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Company_Wallet->>RP_Portal: present the attestations
    RP_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
```

### 1.7. Additionally KYS Information - relevant in Screening process (this will be handled in the MVP+)
```mermaid
sequenceDiagram
    actor Initiator
    RP_Portal<<->>Bank_Wallet: generate proof-request
    RP_Portal<<->>Bank_Wallet: in case that the company required screening : TFS
    RP_Portal<<->>Bank_Wallet: in case that the company has ESG information :  ESG 
    alt Automatically (EUBW support end-points)
        RP_Portal->>+Company_Wallet: request presentations
    else Manually ( EUBW or EUDI Wallet)
        RP_Portal->>RP_Portal: embed request into QRCode and provide an openid4vp-URI link for the request
        Initiator->>+Company_Wallet: copy/paste openid4vp-URI link into the company wallet or scan the QRCode
    end                 
    Company_Wallet<<->>Company_Wallet: mutual authentification ( x509 certificate or eubwoid rulebook)
    Company_Wallet<<->>Company_Wallet: check the authorization of requester to present requested attestations (own bussiness configuration)
    Company_Wallet->>RP_Portal: present the attestations
    RP_Portal<<->>Bank_Wallet: verification of attestations (rulebook)
```

### 1.8. Cross-Check  
```mermaid
sequenceDiagram
    RP_Portal<<->>RP_Portal: cross check over all attestations (to be defined exactly what will be checked)
    RP_Portal->>+Bank_InternalSystem: transfer data to internal system

    RP_Portal<<->>RP_Portal: Display success notification 
```
