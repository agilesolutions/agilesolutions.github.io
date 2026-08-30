# Project Authentication Service

This repository manages the microservice responsible for handling user secure sessions. Below is the system interaction blueprint for the primary login flow.

## User Authentication Sequence

When a user attempts to log into the application, the mobile client verifies credentials securely against our centralized auth gateway using the following chronological lifecycle:

```mermaid
C4Container
    title Container diagram for Internet Banking System

    Person(customer, "Banking Customer", "A customer of the bank, with personal bank accounts")
    System_Ext(email_system, "E-Mail System", "The internal Microsoft Exchange system")

    Container_Boundary(c1, "Internet Banking") {
        Container(web_app, "Web Application", "JavaScript, React", "Delivers the static content and the SPA")
        Container(spa, "Single-Page App", "JavaScript, React", "Provides all banking functionality via the browser")
        Container(mobile_app, "Mobile App", "C#, Xamarin", "Provides a subset of banking functionality")
        ContainerDb(database, "Database", "SQL Database", "Stores user registration, hashed auth credentials, access logs")
        Container(backend_api, "API Application", "Java, Docker", "Provides banking functionality via JSON/HTTPS API")
    }

    Rel(customer, web_app, "Uses", "HTTPS")
    Rel(customer, spa, "Uses", "HTTPS")
    Rel(customer, mobile_app, "Uses")
    Rel(web_app, spa, "Delivers")
    Rel(spa, backend_api, "Makes API calls to", "JSON/HTTPS")
    Rel(mobile_app, backend_api, "Makes API calls to", "JSON/HTTPS")
    Rel(backend_api, database, "Reads from and writes to", "JDBC")
    Rel(email_system, customer, "Sends e-mails to")
    Rel(backend_api, email_system, "Sends e-mails using", "SMTP")
```

```mermaid
sequenceDiagram
    autonumber
    actor User as Mobile App User
    participant App as Mobile Client
    participant API as Auth Gateway
    participant DB as User Database

    User->>App: Input Username & Password
    App->>API: Secure POST /v1/auth/login
    
    activate API
    API->>DB: Query account records
    DB-->>API: Return password hash match
    
    alt Credentials Valid
        API-->>App: 200 OK (Return JWT Token)
        App-->>User: Navigate to Dashboard
    else Credentials Invalid
        API-->>App: 401 Unauthorized Error
        App-->>User: Display "Invalid Password" Banner
    end
    deactivate API
```

## How to Edit This Diagram
If you are using **IntelliJ IDEA** with the **Mermaid Visualizer** plugin installed:
1. Open this `../README.md` file.
2. Click the **Split View** (Editor and Preview) button in the top-right corner of the IDE.
3. Modify the plain-text lines inside the ` ```mermaid ` code fence. The graphical sequence chart will update instantly on your screen.
