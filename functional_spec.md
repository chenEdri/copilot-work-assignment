# Portuguese Tax Authority Transport Documents Web Service Specification

## 1. Overview

This specification describes the integration with the Portuguese Tax Authority's (AT) transport documents web service. This service allows businesses to register electronic transport documents for goods in transit, complying with Portuguese fiscal regulations.

## 2. Service Endpoints

### 2.1 Test Environment
- URL: `https://servicos.portaldasfinancas.gov.pt:701/sgdtws/documentosTransporte`
- Port: 701

### 2.2 Production Environment
- URL: `https://servicos.portaldasfinancas.gov.pt:401/sgdtws/documentosTransporte`
- Port: 401

## 3. Authentication

### 3.1 Client Certificate Authentication
- Client authentication requires X.509 certificates in PKCS#12 (.pfx) format
- TLS 1.2 or higher must be used for all secure connections
- (For Priority - use TestesWebServices.pfx. The password is "TESTEwebservice")

### 3.2 WS-Security Headers

Header structure:
```xml
<S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/">
    <S:Header>
        <wss:Security xmlns:wss="http://schemas.xmlsoap.org/ws/2002/12/secext">
            <wss:UsernameToken>
                <wss:Username>{TaxRegistrationNumber}/{BranchID}</wss:Username>
                <wss:Password>{AES-Encrypted-Password}</wss:Password>
                <wss:Nonce>{RSA-Encrypted-Symmetric-Key}</wss:Nonce>
                <wss:Created>{Encrypted-ISO8601-Timestamp}</wss:Created>
            </wss:UsernameToken>
        </wss:Security>
    </S:Header>
    <S:Body>
        <ns2:envioDocumentoTransporteRequestElem xmlns:ns2="https://servicos.portaldasfinancas.gov.pt/sgdtws/documentosTransporte/">
            <!-- Data fields -->
        </ns2:envioDocumentoTransporteRequestElem>
    </S:Body>
</S:Envelope>
```

Notes:
- AT's public certificate `ChaveCifraPublicaAT2025.cer` must be used for RSA encryption
- Nonce field must contain an RSA-encrypted symmetric key using this certificate
- For Priority, the username and password are: "770005616/1" and "T3QDSV54RK9H"
- Password must be encrypted using AES-128 in ECB mode with PKCS7 padding
- Created timestamp field must be in ISO-8601 format and encrypted using the same scheme as the password

## 4. Request Format

### 4.1 Protocol
- SOAP 1.1
- XML encoding: UTF-8

### 4.2 HTTP Headers
- Content-Type: `text/xml; charset=utf-8`
- SOAPAction: Empty string (`""`)

### 4.3 SOAP Envelope Structure


## 5. Data Structures

### 5.1 Transport Document Fields (Required)

| Field | Type | Description |
|-------|------|-------------|
| TaxRegistrationNumber | String | Company NIF (tax ID) |
| CompanyName | String | Company legal name |
| CompanyAddress | Address | Company address |
| DocumentNumber | String | Unique document reference number |
| MovementStatus | String | Movement status (N=New) |
| MovementDate | Date | YYYY-MM-DD format |
| MovementType | String | Document type (GT=Transport Guide) |
| CustomerTaxID | String | Customer's tax ID |
| CustomerName | String | Customer's name |
| CustomerAddress | Address | Customer's address |
| AddressFrom | Address | Pickup location |
| AddressTo | Address | Delivery location |
| MovementStartTime | DateTime | Format: YYYY-MM-DDThh:mm:ss |
| MovementEndTime | DateTime | Format: YYYY-MM-DDThh:mm:ss |
| VehicleID | String | License plate |
| Line | Array | List of products in transport |

### 5.2 Address Structure

| Field | Type | Description |
|-------|------|-------------|
| Addressdetail | String | Street address |
| City | String | City/locality |
| PostalCode | String | Postal code |
| Country | String | Country code (PT for Portugal) |

### 5.3 Line Item Structure

| Field | Type | Description |
|-------|------|-------------|
| ProductDescription | String | Description of the product |
| Quantity | Decimal | Amount being transported |
| UnitOfMeasure | String | Unit of measurement (UN=Units) |
| UnitPrice | Decimal | Unit price |

## 6. Business Rules and Validation

### 6.1 Document Requirements
- Each document must have a unique DocumentNumber
- MovementDate must be a valid date
- MovementType must be one of the valid document types (GT for Transport Guide)
- MovementStatus must be "N" for new documents
- MovementStartTime must be before MovementEndTime
- At least one Line item must be included

### 6.2 Address Requirements
- Portuguese addresses must include a valid postal code format (XXXX-XXX)
- Country code must be a valid ISO country code (PT for Portugal)

### 6.3 Time Synchronization
- Request timestamps should be synchronized with official NTP servers
- Recommended NTP server: ntp02.oal.ul.pt

## 7. Security Considerations

### 7.1 Encryption
- Password encryption uses AES-128 ECB with PKCS7 padding
- Symmetric key for password encryption must be randomly generated
- That symmetric key is then encrypted with RSA using AT's public certificate
- Base64 encoding is used for binary data

### 7.2 Certificate Management
- Client certificates expire and must be renewed periodically
- AT's public key certificate is updated annually (note the year in filename)

## 8. Response Format

The service will respond with a SOAP envelope containing an acknowledgment or error details. Success responses include a document acceptance confirmation with a unique identifier from AT's system.

## 9. Error Handling

Error responses will include detailed error codes and descriptions. Common errors include validation failures, authentication issues, or service availability problems.

## 10. Sample Request

A complete example of a valid request XML is available in the WSRequest.xml file, which demonstrates all required fields properly formatted.
