# SyntropAI MCP Ecosystem - Architecture Diagrams

## System Context (C4 Level 1)

```mermaid
graph TB
    subgraph "External Systems"
        AWS[AWS Services]
        Azure[Azure Services] 
        OCI[Oracle Cloud Services]
        Finviz[Finviz Financial Data]
    end
    
    subgraph "SyntropAI MCP Ecosystem"
        Client[Claude Desktop / LLM Client]
        MCPServers[MCP Servers]
        SyntropAI[SyntropAIBox Core Library]
    end
    
    Client -->|MCP Protocol| MCPServers
    MCPServers -->|Uses| SyntropAI
    MCPServers -->|SDK Calls| AWS
    MCPServers -->|SDK Calls| Azure
    MCPServers -->|SDK Calls| OCI
    MCPServers -->|API Calls| Finviz
    
    style SyntropAI fill:#e1f5fe
    style MCPServers fill:#f3e5f5
    style Client fill:#e8f5e8
```

## Container Diagram (C4 Level 2)

```mermaid
graph TB
    subgraph "Client Layer"
        Claude[Claude Desktop]
        IDE[IDE Extensions]
        Custom[Custom LLM Apps]
    end
    
    subgraph "MCP Server Layer"
        AWSServer[AWS MCP Server - Docker Container]
        AzureServer[Azure MCP Server - Docker Container]
        OCIServer[OCI MCP Server - Docker Container]
        FinvizServer[Finviz MCP Server - Docker Container]
    end
    
    subgraph "Core Abstraction Layer"
        BaseQuerier[BaseQuerier - Secure Code Execution]
        BaseSession[BaseSession - Provider Authentication]
        Sandbox[AST Sandbox - Security Validation]
        SyntropAIBox[(SyntropAIBox Library - Published on TestPyPI)]
    end
    
    subgraph "Cloud SDKs"
        Boto3[boto3 SDK]
        AzureSDK[Azure Python SDK]
        OCISDK[OCI Python SDK]
        FinvizSDK[finvizfinance]
    end
    
    Claude -->|stdio/JSON-RPC| AWSServer
    IDE -->|stdio/JSON-RPC| AzureServer
    Custom -->|stdio/JSON-RPC| OCIServer
    Claude -->|stdio/JSON-RPC| FinvizServer
    
    AWSServer --> BaseQuerier
    AzureServer --> BaseQuerier
    OCIServer --> BaseQuerier
    FinvizServer --> BaseQuerier
    
    BaseQuerier --> BaseSession
    BaseQuerier --> Sandbox
    BaseQuerier --> SyntropAIBox
    
    AWSServer --> Boto3
    AzureServer --> AzureSDK
    OCIServer --> OCISDK
    FinvizServer --> FinvizSDK
    
    style SyntropAIBox fill:#e1f5fe
    style BaseQuerier fill:#fff3e0
    style Sandbox fill:#ffebee
```

## Component Architecture

```mermaid
graph TB
    subgraph "SyntropAIBox Core Components"
        subgraph "Base Classes"
            BQ[BaseQuerier - execute_query, build_schema, run_code_tool]
            BS[BaseSession - from_args, configure_parser]
        end
        
        subgraph "Security Layer"
            CE[CodeExecutor - AST Transformer, visit_Import, visit_Call]
            SB[create_safe_builtins - Whitelist Functions]
            TO[exec_with_timeout - Timeout Protection]
        end
        
        subgraph "Execution Environment"
            NS[Namespace Injection - SDK Objects & Credentials]
            AM[Allowed Modules - Security Whitelist]
            ER[Error Handling - JSON Serialization]
        end
    end
    
    BQ --> CE
    BQ --> SB
    BQ --> TO
    BQ --> NS
    BQ --> AM
    BQ --> ER
    
    style BQ fill:#e3f2fd
    style BS fill:#e8f5e8
    style CE fill:#ffebee
    style SB fill:#fff3e0

```

## MCP Protocol Flow

```mermaid
sequenceDiagram
    participant C as Claude Desktop
    participant MS as MCP Server
    participant BQ as BaseQuerier
    participant SDK as Cloud SDK
    participant CS as Cloud Service
    
    C->>MS: Initialize MCP Connection
    MS->>C: Server Capabilities
    
    C->>MS: list_tools()
    MS->>C: Available Tools Schema
    
    C->>MS: call_tool(code_snippet)
    MS->>BQ: execute_query(code)
    
    BQ->>BQ: Parse AST & Validate
    BQ->>BQ: Check Allowed Imports
    BQ->>BQ: Create Safe Namespace
    
    BQ->>SDK: Execute Code with SDK
    SDK->>CS: API Call
    CS->>SDK: Response Data
    SDK->>BQ: Processed Result
    
    BQ->>MS: JSON Serialized Result
    MS->>C: Tool Response
```

## Security Architecture

```mermaid
graph TB
    subgraph "User Input"
        UI["Code Snippet from User"]
    end
    
    subgraph "Security Validation Pipeline"
        AST["AST Parser - Python ast.parse"]
        VAL["Code Validator - visit_Call, visit_Import"]
        WL["Whitelist Check - Allowed Modules & Functions"]
        NS["Safe Namespace - Controlled builtins"]
        TO["Timeout Protection - Signal alarm handler"]
    end
    
    subgraph "Execution Environment"
        EXEC["Sandbox Exec (exec)"]
        RESULT["Capture result variable"]
    end
    
    subgraph "Output Processing"
        SER["JSON Serialization"]
        ERR["Error Handling"]
    end
    
    UI --> AST
    AST --> VAL
    VAL -->|Valid| WL
    VAL -->|Invalid| ERR
    WL -->|Approved| NS
    WL -->|Blocked| ERR
    NS --> TO
    TO --> EXEC
    EXEC --> RESULT
    RESULT --> SER
    SER --> ERR
    
    style VAL fill:#ffebee
    style WL fill:#fff3e0
    style TO fill:#f3e5f5
    style ERR fill:#fce4ec

```

## Multi-Cloud Abstraction Pattern

```mermaid
graph TB
    subgraph "Unified Interface Layer"
        UI[User Interface - Single API Pattern]
    end
    
    subgraph "Provider Abstraction"
        AWS_S[AWSSession - boto3.Session]
        Azure_S[AzureSession - DefaultAzureCredential]
        OCI_S[OCISession - oci.config]
        
        AWS_Q[AWSResourceQuerier - BaseQuerier Implementation]
        Azure_Q[AzureResourceQuerier - BaseQuerier Implementation] 
        OCI_Q[OCIResourceQuerier - BaseQuerier Implementation]
    end
    
    subgraph "Provider SDKs"
        Boto3[boto3 - AWS Python SDK]
        AzureSDK[azure-* - Azure Python SDKs]
        OCISDK[oci - Oracle Cloud SDK]
    end
    
    subgraph "Cloud Services"
        AWS_Services[AWS Services - EC2, S3, Lambda, etc.]
        Azure_Services[Azure Services - VMs, Storage, Functions, etc.]
        OCI_Services[OCI Services - Compute, Object Storage, etc.]
    end
    
    UI --> AWS_Q
    UI --> Azure_Q  
    UI --> OCI_Q
    
    AWS_Q --> AWS_S
    Azure_Q --> Azure_S
    OCI_Q --> OCI_S
    
    AWS_S --> Boto3
    Azure_S --> AzureSDK
    OCI_S --> OCISDK
    
    Boto3 --> AWS_Services
    AzureSDK --> Azure_Services
    OCISDK --> OCI_Services
    
    style UI fill:#e8f5e8
    style AWS_Q fill:#fff3e0
    style Azure_Q fill:#e3f2fd
    style OCI_Q fill:#f3e5f5
```

## Data Flow Architecture

```mermaid
flowchart TD
    subgraph "Input Layer"
        UC[User Code Snippet]
        AUTH[Authentication Config]
    end
    
    subgraph "Processing Pipeline"
        PARSE[AST Parsing & Validation]
        AUTH_SETUP[Session Authentication]
        NS_CREATE[Namespace Creation]
        EXEC[Safe Code Execution]
        RESULT_EXTRACT[Result Extraction]
    end
    
    subgraph "Output Layer"
        JSON[JSON Serialization]
        ERROR[Error Handling]
        RESPONSE[MCP Response]
    end
    
    subgraph "External Integrations"
        SDK_CALL[SDK Method Calls]
        API_RESP[Cloud API Responses]
    end
    
    UC --> PARSE
    AUTH --> AUTH_SETUP
    
    PARSE -->|Valid| NS_CREATE
    PARSE -->|Invalid| ERROR
    
    AUTH_SETUP --> NS_CREATE
    NS_CREATE --> EXEC
    
    EXEC --> SDK_CALL
    SDK_CALL --> API_RESP
    API_RESP --> RESULT_EXTRACT
    
    RESULT_EXTRACT --> JSON
    JSON --> RESPONSE
    ERROR --> RESPONSE
    
    style PARSE fill:#ffebee
    style EXEC fill:#e8f5e8  
    style ERROR fill:#fce4ec
    style SDK_CALL fill:#e3f2fd
```

## Deployment Architecture

```mermaid
graph TB
    subgraph "Development Environment"
        DEV[Local Development]
        TEST[Testing & Validation]
    end
    
    subgraph "Container Registry"
        DOCKER[Docker Images - mcp-server-*]
    end
    
    subgraph "Distribution"
        PYPI[TestPyPI - syntropaibox package]
        GIT[Git Repositories - Individual MCP Servers]
    end
    
    subgraph "Runtime Environment"
        CONTAINER[Docker Containers - Isolated Execution]
        CONFIG[Configuration Files - claude_config.json]
        CLIENT[MCP Clients - Claude Desktop, IDEs]
    end
    
    subgraph "Cloud Authentication"
        AWS_CRED[AWS Credentials - ~/.aws/]
        AZURE_CRED[Azure Credentials - Environment Variables]
        OCI_CRED[OCI Credentials - ~/.oci/config]
    end
    
    DEV --> TEST
    TEST --> DOCKER
    TEST --> PYPI
    TEST --> GIT
    
    DOCKER --> CONTAINER
    PYPI --> CONTAINER
    CONFIG --> CLIENT
    CONTAINER --> CLIENT
    
    CONTAINER --> AWS_CRED
    CONTAINER --> AZURE_CRED
    CONTAINER --> OCI_CRED
    
    style CONTAINER fill:#e8f5e8
    style PYPI fill:#e3f2fd
    style CLIENT fill:#fff3e0
```

## Key Architectural Principles

### 1. **Abstraction Layers**
- **Interface Layer**: Consistent MCP protocol implementation
- **Business Logic**: Provider-agnostic query execution
- **Infrastructure Layer**: Provider-specific SDK integration

### 2. **Security by Design**
- **Input Validation**: AST-based code analysis
- **Execution Isolation**: Sandboxed environments
- **Access Control**: Whitelisted modules and functions

### 3. **Extensibility**
- **Plugin Architecture**: Easy addition of new providers
- **Configuration-Driven**: Runtime behavior modification
- **Schema Generation**: Dynamic API documentation

### 4. **Resilience**
- **Error Boundaries**: Comprehensive exception handling
- **Timeout Protection**: Resource usage limits
- **Graceful Degradation**: Partial failure handling

These diagrams illustrate how the SyntropAI MCP ecosystem provides a unified, secure, and extensible platform for multi-cloud resource management through intelligent abstraction and robust security mechanisms.