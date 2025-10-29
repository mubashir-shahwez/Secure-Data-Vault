# 🔐 Secure Data Vault

A desktop application for securely storing sensitive text secrets and encrypted files using AES-256 encryption, PBKDF2 password hashing, and MySQL persistence.

## ✨ Features
- User authentication with PBKDF2-HMAC-SHA256 (salted, 100k iterations)
- AES‑256‑CBC encryption for secrets and files (random IV per operation)
- Secrets CRUD, search, timestamps, decrypt‑on‑demand
- File encryption with 10 MB cap, image/text in‑app preview, external open for others
- Proof of encryption: "View Cipher" (secrets) and "File Cipher" (files)
- Access logging for VIEW/ADD/UPDATE/DELETE/DOWNLOAD_FILE

## 🛠 Technology Stack
- Java 17+, Swing UI
- MySQL 8.0+, JDBC (MySQL Connector/J)
- JCE crypto (AES‑256‑CBC, PBKDF2‑HMAC‑SHA256)

## 🔒 Security Architecture
- Passwords: stored as "salt:hash" (Base64) in VARCHAR(512)
- Key derivation: PBKDF2‑HMAC‑SHA256 (100k iterations) → 256‑bit AES key
- Encryption: AES‑256‑CBC with random 16‑byte IV; IV is stored alongside ciphertext
- Secrets: Base64(IV||ciphertext) in DB; Files: LONGBLOB (IV||ciphertext)
- Decrypt‑on‑demand only; plaintext never stored at rest

## 📦 Prerequisites
- JDK 17+
- MySQL 8.0+
- MySQL Connector/J 8.0+ (JDBC driver)

## 🚀 Setup

### 1) Clone
git clone https://github.com/your-username/DataSecureVault.git
cd DataSecureVault

text

### 2) Create Database
CREATE DATABASE vaultdb;
USE vaultdb;

CREATE TABLE users (
id INT AUTO_INCREMENT PRIMARY KEY,
username VARCHAR(50) NOT NULL UNIQUE,
password_hash VARCHAR(512) NOT NULL,
salt VARCHAR(255) NULL,
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
last_login TIMESTAMP NULL,
INDEX idx_username (username)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE vault_data (
id INT AUTO_INCREMENT PRIMARY KEY,
user_id INT NOT NULL,
key_name VARCHAR(100) NOT NULL,
secret_value TEXT NOT NULL,
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
INDEX idx_user_key (user_id, key_name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE vault_files (
id INT AUTO_INCREMENT PRIMARY KEY,
user_id INT NOT NULL,
file_name VARCHAR(255) NOT NULL,
file_type VARCHAR(50) NOT NULL,
file_size INT NOT NULL,
encrypted_data LONGBLOB NOT NULL,
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
INDEX idx_user_filename (user_id, file_name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE access_logs (
id INT AUTO_INCREMENT PRIMARY KEY,
user_id INT NOT NULL,
action VARCHAR(50) NOT NULL,
key_name VARCHAR(100) NULL,
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
INDEX idx_user_action (user_id, action)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

text

### 3) Configure DB
Create file: `src/com/vault/config.properties`
db.type=mysql
db.url=jdbc:mysql://localhost:3306/vaultdb
db.username=root
db.password=your_mysql_password

text
Add `config.properties` to `.gitignore`.

### 4) Run
- IntelliJ: open project → Run `Main.java`
- CLI example:
javac -d out -sourcepath src src/com/vault/ui/Main.java
java -cp "out:lib/mysql-connector-j-8.0.xx.jar" com.vault.ui.Main

text

## 📖 Usage
- Register → Login with master password
- Secrets:
  - Add secret → View Secret (decrypt) → View Cipher (Base64 ciphertext) → Update/Delete
- Files:
  - Add File (≤ 10 MB) → Preview File (images/text in‑app) → File Cipher (Base64) → Download
- Search: type in search box → click 🔎 to filter by key name

## 📂 Structure
src/com/vault/
core/ (DatabaseManager, EncryptionManager)
model/ (User, Secret, AccessLog)
service/ (UserService, VaultService, VaultFileService)
ui/ (Main, LoginFrame, MainVaultFrame,
AddSecretDialog, UpdateSecretDialog,
ViewSecretDialog, ViewCipherDialog)

text

## ⚠️ Security Notes
- CBC provides confidentiality; for integrity, consider AES‑GCM in future
- Master password is critical; losing it means data is unrecoverable
- No key rotation yet; changing master will require re‑encryption
- Use strong passwords and secure DB access (least privilege, TLS)

## 🧭 Future Enhancements
- AES‑GCM (authenticated encryption)
- Master password rotation
- PDF inline preview (PDFBox)
- 2FA (TOTP), rate limiting, lockouts
- Export/import encrypted backups
## 📄 License
MIT License
