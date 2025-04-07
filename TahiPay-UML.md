# TahiPay System Workflows

  
## Common Payment Flow


```mermaid

sequenceDiagram

participant Customer

participant Merchant

participant Client

participant API

participant Blockchain

  

Customer->>Merchant: Scan QR code

Merchant->>Client: Get order details

Client->>API: GET /merchants/{id}/orders/{id}

API-->>Client: Return order info

Client->>Customer: Show payment amount

Customer->>Blockchain: Approve payment

Customer->>API: POST /orders/{id}/approve

API->>Blockchain: Verify approval

Blockchain-->>API: Confirmation

API-->>Client: Payment approved

Client->>API: POST /orders/{id}/collect-payment

API->>Blockchain: Transfer tokens

Blockchain-->>API: Transaction hash

API-->>Client: Payment completed

```

  

## Gasless Payment Flow

  

```mermaid

sequenceDiagram

participant Customer

participant Merchant

participant Client

participant API

participant Blockchain

  

Customer->>Merchant: Scan QR code

Merchant->>Client: Get order details

Client->>API: GET /merchants/{id}/orders/{id}

API-->>Client: Return order info

Client->>API: GET /gasless/token-info/{token}

API-->>Client: Return token info

Client->>API: GET /gasless/get-nonce/{token}/{wallet}

API-->>Client: Return nonce

Client->>Customer: Sign permit

Customer->>API: POST /orders/{id}/pay-gasless

API->>Blockchain: Submit permit + transfer

Blockchain-->>API: Transaction hash

API-->>Client: Payment completed

```


## Authentication Flow

  

```mermaid

sequenceDiagram

participant User

participant Client

participant API

participant AuthService

  

User->>Client: Enter credentials

Client->>API: POST /auth/login

API->>AuthService: Validate credentials

AuthService-->>API: Generate JWT token

API-->>Client: Return token + refresh token

Client->>User: Store token

Note over Client: Token used for subsequent requests

```

  


  

## Product Management Flow

  

```mermaid

sequenceDiagram

participant Merchant

participant Client

participant API

participant DB

  

Merchant->>Client: Add product details

Client->>API: POST /merchants/{id}/products

API->>DB: Create product record

DB-->>API: Return product data

API-->>Client: Return product details

Client->>Merchant: Show product added

```

  

## Order Creation Flow

  

```mermaid

sequenceDiagram

participant Merchant

participant Client

participant API

participant DB

  

Merchant->>Client: Select products

Client->>API: POST /merchants/{id}/orders

API->>DB: Create order record

DB-->>API: Return order data

API-->>Client: Return order + QR code

Client->>Merchant: Display QR code

```

  

  

## Balance Transfer Flow

  

```mermaid

sequenceDiagram

participant Merchant

participant Client

participant API

participant Blockchain

  

Merchant->>Client: Enter transfer details

Client->>API: POST /merchants/{id}/balances

API->>Blockchain: Verify balance

Blockchain-->>API: Balance confirmed

API->>Blockchain: Transfer tokens

Blockchain-->>API: Transaction hash

API-->>Client: Transfer completed

Client->>Merchant: Show success message

```

  

## System Components

  

```mermaid

classDiagram

class Merchant {

+UUID id

+String name

+String email

+String walletAddress

+String apiKey

+createOrder()

+updateProfile()

+transferBalance()

}

  

class Product {

+Integer id

+UUID merchantId

+String name

+Float price

+Boolean inStock

}

  

class Order {

+UUID id

+UUID merchantId

+String status

+Float totalAmount

+String paymentTxHash

+createQRCode()

+approvePayment()

+collectPayment()

}

  

class Payment {

+String txHash

+String sender

+String recipient

+String amount

+processGasless()

+processStandard()

}

  

Merchant "1" -- "many" Product

Merchant "1" -- "many" Order

Order "1" -- "1" Payment

```

  

## State Transitions

  

```mermaid

stateDiagram-v2

[*] --> Pending

Pending --> Approved: Customer approves

Approved --> Paid: Merchant collects

Pending --> Cancelled: Merchant cancels

Approved --> Cancelled: Merchant cancels

Paid --> [*]

Cancelled --> [*]

```

  

## Security Flow

  

```mermaid

sequenceDiagram

participant Client

participant API

participant AuthService

participant Blockchain

  

Client->>API: Request with JWT

API->>AuthService: Validate token

AuthService-->>API: Token valid

API->>Blockchain: Verify wallet ownership

Blockchain-->>API: Ownership confirmed

API-->>Client: Process request

```

  

## WebSocket Notifications

  

```mermaid

sequenceDiagram

participant Client

participant API

participant WebSocket

participant Blockchain

  

Client->>API: Request with clientId

API->>WebSocket: Register client

Blockchain->>API: Transaction event

API->>WebSocket: Push notification

WebSocket->>Client: Update status

```