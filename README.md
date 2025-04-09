# copilot-work-assignment
Work assignment using copilot.

# AT Document Transport

A utility for securely submitting documents to the Portuguese Tax Authority (AT).

## Features

- WS-Security compliant SOAP message generation
- RSA encryption of nonce using AT's public certificate
- AES-128-ECB encryption for passwords and timestamps
- XML document processing with proper security headers
- Send documents directly to AT test or production environments

## Usage

```shell
# Generate SOAP with security headers
./atdocument -username user123 -password SecurePass123

# Send document to AT test environment
./atdocument -username user123 -password SecurePass123 -send -test

# Send document to AT production environment
./atdocument -username user123 -password SecurePass123 -send

# With explicit paths
./atdocument -username user123 -password SecurePass123 -cert /path/to/cert.cer -xml /path/to/input.xml -output /path/to/output.xml -send
```

### Using Client Certificate Authentication

```shell
# With default client certificate in current directory
./atdocument -username user123 -password SecurePass123 -cert-password certPassword -send

# With explicit client certificate path
./atdocument -username user123 -password SecurePass123 -client-cert /path/to/cert.pfx -cert-password certPassword -send
```

### Using PEM Certificate Files

If you have certificate and key in separate PEM files (common in Linux/Unix environments), you can use them directly:

```shell
# Using cert.pem and key.pem files
./atdocument -username user123 -password SecurePass123 -cert-pem cert.pem -key-pem key.pem -send
```

This approach doesn't require any conversion and works with standard PEM format certificate and key files.

## Command Line Options

- `-username` - Username for authentication (required)
- `-password` - Password to encrypt (required)
- `-cert` - Path to AT public certificate (uses ChaveCifraPublicaAT2025.cer if not specified)
- `-xml` - Path to XML file for SOAP body (uses WSRequest.xml if not specified)
- `-output` - Output file path for generated SOAP (generates name based on timestamp if not specified)
- `-test` - Use the test environment instead of production
- `-send` - Send the document to AT after generating it
- `-tls12` - Use TLS 1.2 only (can help resolve TLS connection issues)
- `-compat` - Use maximum compatibility mode for difficult networks (slowest but most reliable)
- `-debug` - Enable additional debug logging
- `-client-cert` - Path to client certificate .pfx file for authentication (uses TestesWebServices.pfx if not specified)
- `-cert-password` - Password for client certificate (required when using client certificate)
- `-cert-pem` - Path to client certificate PEM file (e.g., cert.pem)
- `-key-pem` - Path to client private key PEM file (e.g., key.pem)

## Default Files

The application will look for these files in the current directory if not explicitly specified:

- `ChaveCifraPublicaAT2025.cer` - Default AT public certificate
- `WSRequest.xml` - Default XML request template
- `TestesWebServices.pfx` - Default client certificate

## Security Requirements

- Username must be at least 3 characters
- Password must be at least 8 characters
- The AT public certificate must be valid

## Client Certificate Authentication

The application supports client certificate authentication (mutual TLS) when communicating with the AT service.
To use this feature:

1. Ensure you have a valid PKCS#12 (.pfx) certificate file
2. Use the `-client-cert` flag to specify the certificate file path
3. Provide the certificate password with the `-cert-password` flag

The application will look for a file named `TestesWebServices.pfx` in the current directory if no certificate path is provided.

## Installation

1. Download the latest release binary
2. Ensure the default certificate is in the same directory or specify its path
3. Create or customize your XML document based on the template

For developers:

```shell
go build -o atdocument
```

## Response Handling

When sending documents, the response from AT will be:
1. Displayed in the console
2. Saved to a file with ".response.xml" appended to the output filename

Successful responses will include a return code of "0000" and may include an ATCUD code.

## Troubleshooting TLS Issues

If you encounter the error "local error: tls: bad record MAC" when sending documents, try these steps:

1. First, try using the `-tls12` flag to force TLS 1.2 only:
   ```
   ./atdocument -username user123 -password SecurePass123 -send -tls12
   ```

2. If that doesn't work, try the maximum compatibility mode:
   ```
   ./atdocument -username user123 -password SecurePass123 -send -compat
   ```

3. Check your network configuration:
   - Make sure no proxies are interfering with the SSL/TLS connection
   - Verify firewall settings allow secure connections to the AT endpoints
   - Disable any SSL/TLS inspection in your firewall or proxy

4. Verify you're using the correct endpoints:
   - Production: https://servicos.portaldasfinancas.gov.pt/sgdtws/documentosTransporte
   - Test: https://servicos.portaldasfinancas.gov.pt:701/sgdtws/documentosTransporte

5. If you still have issues, it may be helpful to test directly with curl:
   ```
   curl -v -k -X POST https://servicos.portaldasfinancas.gov.pt:701/sgdtws/documentosTransporte
   ```

6. Ensure you have the latest version of the AT certificate (ChaveCifraPublicaAT2025.cer)

7. If all else fails, contact the AT support team for assistance with their connectivity requirements.

## Troubleshooting Certificate Issues

If you encounter the error "failed to parse P12 file: pkcs12: expected exactly two safe bags in the PFX PDU" when using client certificate authentication, it indicates an issue with your PKCS#12 (.pfx) certificate format.

### Fix with the Conversion Script

The simplest solution is to convert your certificate using the provided script:

```bash
# On Linux/Mac
./scripts/convert_certificate.sh TestesWebServices.pfx your_password fixed_certificate.pfx

# On Windows
bash scripts/convert_certificate.sh TestesWebServices.pfx your_password fixed_certificate.pfx
```

Then use the converted certificate:

```bash
./atdocument -username user123 -password SecurePass123 -client-cert fixed_certificate.pfx -cert-password your_password -send
```

### Fix "Non-standard PKCS#12 format detected, trying alternative Go-only methods"

If you see this error message, it means your certificate is in a format that Go's standard libraries cannot process correctly. The most reliable fix is to convert your certificate to a standard format:

#### Using the Provided Scripts

**On Linux/Mac:**
```bash
./scripts/convert_certificate.sh TestesWebServices.pfx your_password fixed_certificate.pfx
```

**On Windows:**
```cmd
scripts\convert_certificate.bat TestesWebServices.pfx your_password fixed_certificate.pfx
```

Then use the converted certificate:
```bash
./atdocument -username user123 -password SecurePass123 -client-cert fixed_certificate.pfx -cert-password your_password -send
```

#### Prerequisites

- OpenSSL must be installed and available in your PATH
- For Windows users: You can download OpenSSL from https://slproweb.com/products/Win32OpenSSL.html

#### If the Script Doesn't Work

1. Make sure OpenSSL is installed correctly:
   ```bash
   openssl version
   ```

2. Try manually converting your certificate:
   ```bash
   # Extract private key
   openssl pkcs12 -in TestesWebServices.pfx -nocerts -out key.pem -passin pass:your_password -passout pass:your_password
   
   # Extract certificates
   openssl pkcs12 -in TestesWebServices.pfx -nokeys -out cert.pem -passin pass:your_password
   
   # Create new PKCS#12 file
   openssl pkcs12 -export -out fixed_certificate.pfx -inkey key.pem -in cert.pem -passin pass:your_password -passout pass:your_password
   
   # Clean up
   rm key.pem cert.pem
   ```

3. You can also try exporting the certificate again from your certificate management tool:
   - Choose PKCS#12 or PFX format
   - Include the private key
   - Use a simple password (avoid special characters)
   - Select the option "Microsoft Enhanced RSA and AES Cryptographic Provider" if available

### Manual Fix using OpenSSL

If the script doesn't work, you can perform the steps manually:

```bash
# 1. Extract the private key
openssl pkcs12 -in TestesWebServices.pfx -nocerts -out key.pem -passin pass:your_password -passout pass:your_password

# 2. Extract the certificates
openssl pkcs12 -in TestesWebServices.pfx -nokeys -out cert.pem -passin pass:your_password

# 3. Create a new PKCS#12 file with the correct format
openssl pkcs12 -export -out fixed_certificate.pfx -inkey key.pem -in cert.pem -passin pass:your_password -passout pass:your_password

# 4. Clean up temporary files
rm key.pem cert.pem
```

### OpenSSL Not Found Error

If you encounter the error "OpenSSL executable file not found in %PATH%", it means that the application 
tried to use OpenSSL as a fallback method for handling problematic certificate formats, but OpenSSL 
is not installed or not in your system PATH.

Solutions:

1. Install OpenSSL:
   - Windows: Download from [https://slproweb.com/products/Win32OpenSSL.html](https://slproweb.com/products/Win32OpenSSL.html)
   - Linux: Use your package manager (e.g., `apt install openssl`, `yum install openssl`)
   - Mac: Use Homebrew (`brew install openssl`)

2. Ensure OpenSSL is in your PATH:
   - Windows: Add the OpenSSL bin directory to your PATH environment variable
   - Linux/Mac: OpenSSL should be in your PATH by default if installed via package manager

3. Use a standard format PKCS#12 certificate:
   - If you have the private key and certificate separately, you can create a properly formatted PKCS#12 file
   - Use a certificate management tool to export the certificate in standard PKCS#12 format

Example of creating a standard PKCS#12 file from separate key and certificate files:

```bash
openssl pkcs12 -export -out standard_cert.pfx -inkey private_key.pem -in certificate.pem -passout pass:your_password
```
