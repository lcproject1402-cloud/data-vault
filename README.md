# Data Vault

**Data Vault** is a highly secure and scalable data storage system for handling sensitive data at an enterprise level. With built-in encryption and redundancy, **Data Vault** ensures that your data is always safe and available, no matter the circumstances. 

## Features
- **End-to-End Encryption:** All data is encrypted at rest and in transit using AES-256.
- **Redundant Storage:** Automatically replicates data across multiple regions to ensure availability.
- **Scalable:** Designed for both small applications and large enterprises.
- **User Access Control:** Supports granular permission settings to control access to sensitive information.
- **API Interface:** Simple REST API to interact with your data, including uploading, downloading, and managing files.

## Installation

Clone the repository:
```bash
git clone https://github.com/yourusername/data-vault.git
cd data-vault

Install the required dependencies:

pip install -r requirements.txt

Set up environment variables for configuration:

export DATA_VAULT_KEY="your_encryption_key"
export DATA_VAULT_REGION="us-east-1"
Usage
Upload Data

Use the upload_data function to securely upload files to the vault.

from data_vault import DataVault

vault = DataVault(api_key="your_api_key")

# Upload a file
vault.upload_data("path/to/your/file.txt")
Download Data

To download your data from the vault, use the download_data function.

vault.download_data("file_id", "downloaded_file.txt")
Get File Information

Get metadata and details about a file stored in the vault.

file_info = vault.get_file_info("file_id")
print(file_info)
License

This project is licensed under the MIT License - see the LICENSE
 file for details.


---

### Code Files

Here’s an outline of the files and code that would be in the repository:

#### `data_vault.py`
```python
import os
import requests
import hashlib
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend

class DataVault:
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.datavault.com/v1"
        self.encryption_key = os.environ.get('DATA_VAULT_KEY')
        self.region = os.environ.get('DATA_VAULT_REGION', 'us-east-1')

    def _encrypt_data(self, data):
        # Encrypt data using AES-256
        key = hashlib.sha256(self.encryption_key.encode()).digest()
        cipher = Cipher(algorithms.AES(key), modes.GCM(self._generate_nonce()), backend=default_backend())
        encryptor = cipher.encryptor()
        encrypted_data = encryptor.update(data.encode()) + encryptor.finalize()
        return encrypted_data

    def _generate_nonce(self):
        # Generate a random nonce
        return os.urandom(12)

    def upload_data(self, file_path):
        with open(file_path, 'rb') as f:
            file_data = f.read()

        encrypted_data = self._encrypt_data(file_data)

        response = requests.post(f"{self.base_url}/upload", headers={
            'Authorization': f'Bearer {self.api_key}',
        }, files={'file': encrypted_data})

        if response.status_code == 200:
            return response.json()
        else:
            return {"error": "Upload failed", "status_code": response.status_code}

    def download_data(self, file_id, output_path):
        response = requests.get(f"{self.base_url}/files/{file_id}", headers={
            'Authorization': f'Bearer {self.api_key}',
        })

        if response.status_code == 200:
            encrypted_data = response.content
            decrypted_data = self._decrypt_data(encrypted_data)

            with open(output_path, 'wb') as f:
                f.write(decrypted_data)
        else:
            return {"error": "Download failed", "status_code": response.status_code}

    def _decrypt_data(self, encrypted_data):
        # Decrypt the data using AES-256
        key = hashlib.sha256(self.encryption_key.encode()).digest()
        cipher = Cipher(algorithms.AES(key), modes.GCM(self._generate_nonce()), backend=default_backend())
        decryptor = cipher.decryptor()
        decrypted_data = decryptor.update(encrypted_data) + decryptor.finalize()
        return decrypted_data

    def get_file_info(self, file_id):
        response = requests.get(f"{self.base_url}/files/{file_id}/info", headers={
            'Authorization': f'Bearer {self.api_key}',
        })

        if response.status_code == 200:
            return response.json()
        else:
            return {"error": "File info retrieval failed", "status_code": response.status_code}
requirements.txt
requests==2.26.0
cryptography==3.4.7
LICENSE

MIT License, simple and open.
