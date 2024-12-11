# **🚀 Comprehensive Guide to iOS Build Credentials with EAS CLI on Windows**

This guide will help you set up and use **custom iOS credentials** (like `.p12` and provisioning profiles) to build and submit an iOS app using `EAS CLI` on Windows. Let's get started! 🌟

---

## **📋 Prerequisites**
1. A **Windows RDP** or VPS with administrative access.
2. An active **Apple Developer Account**.
3. Installed:
   - **Node.js**: Download it from [Node.js Official Site](https://nodejs.org/).
   - **EAS CLI**: Install it globally using:
     ```bash
     npm install -g eas-cli
     ```
4. Access to [Apple Developer Portal](https://developer.apple.com/) and [App Store Connect](https://appstoreconnect.apple.com/).

---

## **1️⃣ Install OpenSSL on Windows**
### **Step 1.1: Download OpenSSL**
- Visit [OpenSSL for Windows](https://slproweb.com/products/Win32OpenSSL.html).
- Download the **light version**:
  - **64-bit**: `Win64 OpenSSL v<version> Light`
  - **32-bit**: `Win32 OpenSSL v<version> Light`

### **Step 1.2: Install OpenSSL**
- Run the downloaded `.exe` file and follow the installation steps.
- Install OpenSSL at the default location: `C:\Program Files\OpenSSL-Win64`.

### **Step 1.3: Add OpenSSL to System Path**
1. Go to the **OpenSSL bin folder**: `C:\Program Files\OpenSSL-Win64\bin`.
2. Copy the path.
3. Add it to your system environment variables:
   - Right-click **This PC** → **Properties** → **Advanced system settings** → **Environment Variables**.
   - Edit the `Path` variable under **System Variables** and paste the path.

### **Step 1.4: Verify Installation**
- Open Command Prompt and type:
  ```bash
  openssl version
  ```
- If installed correctly, you’ll see the OpenSSL version.

---

## **2️⃣ Configure `openssl.cnf` File**
### **Step 2.1: Locate `openssl.cnf`**
- Navigate to `C:\Program Files\OpenSSL-Win64\bin` and locate `openssl.cnf`.

### **Step 2.2: Edit the Configuration**
Replace or update the `[req]` and `[req_distinguished_name]` sections as follows:

```ini
[ req ]
default_bits       = 2048                 # RSA key size
default_keyfile    = mykey.key            # Name of the private key file
distinguished_name = req_distinguished_name  # Links to the distinguished_name section
prompt             = no                   # Disables interactive prompts
output_password    = mypassword           # Optional password for the key

[ req_distinguished_name ]
countryName            = USA               # Country code
stateOrProvinceName    = Calfornia          # State or province
localityName           = Street 1, here   # City or specific address
organizationName       = CompanyName LLC    # Organization name
commonName             = website.com      # Domain or app name
```

### **💡 What's `distinguished_name = req_distinguished_name`?**
It specifies which section (`[req_distinguished_name]`) contains the certificate details (like country, organization, etc.).

---

## **3️⃣ Generate `.p12` and CSR**
### **Step 3.1: Create a Folder**
- Create a folder (e.g., `C:\KeyFolder`) to store your files.

### **Step 3.2: Generate a Private Key**
Run the following command:
```bash
openssl genrsa -out mykey.key 2048
```

### **Step 3.3: Generate a CSR**
Run:
```bash
openssl req -new -key mykey.key -out mycsr.csr -config "C:\Program Files\OpenSSL-Win64\bin\openssl.cnf"
```

### **Step 3.4: Create iOS Distribution Certificate**
1. Log in to [Apple Developer Portal](https://developer.apple.com/).
2. Go to **Certificates, Identifiers & Profiles** → **Certificates** → Add a new **iOS Distribution Certificate**.
3. Upload `mycsr.csr` and download the resulting `distribution.cer`.

### **Step 3.5: Convert `.cer` to `.p12`**
1. Convert `.cer` to `.pem`:
   ```bash
   openssl x509 -in distribution.cer -inform DER -out distribution.pem -outform PEM
   ```
2. Combine `.pem` and private key into `.p12`:
   ```bash
   openssl pkcs12 -export -inkey mykey.key -in distribution.pem -out distributionunity.p12
   ```
3. Set a password for the `.p12` file when prompted.

---

## **4️⃣ Configure EAS with Custom Credentials**
### **Step 4.1: Create `credentials.json`**
In your project root, create a `credentials.json` file with the following content:

```json
{
  "ios": {
    "distributionCertificate": {
      "path": "./credentials/distributionunity.p12",
      "password": "your_p12_password"
    },
    "provisioningProfilePath": "./credentials/provisioning.mobileprovision"
  }
}
```

- Replace the paths with the actual file paths to your `.p12` and provisioning profile.
- Save these files in a `credentials` folder within your project.

### **Step 4.2: Update `eas.json`**
Modify `eas.json` to use local credentials:
```json
{
  "build": {
    "production": {
      "ios": {
        "credentialsSource": "local"
      }
    }
  }
}
```

---

## **5️⃣ Build and Submit the iOS App**
### **Step 5.1: Build with EAS**
Run the iOS build command:
```bash
eas build --platform ios
```

EAS will automatically use the credentials provided in `credentials.json`.

### **Step 5.2: Submit to App Store**
1. Install EAS Submit:
   ```bash
   npm install -g eas-cli
   ```
2. Submit the `.ipa` file:
   ```bash
   eas submit --platform ios
   ```
   - Log in to your Apple Developer account.
   - Follow the prompts to upload your app to App Store Connect.

---

## **🎯 Key Takeaways**
- Use a Windows RDP to set up OpenSSL and generate iOS credentials.
- Keep the `.p12` and provisioning profile files secure.
- `credentials.json` simplifies manual credential management in EAS.
