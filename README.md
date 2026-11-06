# FranchiseOps AI — Agentic Franchise Operations Intelligence Platform

## Milestone 1: User Authentication & Operations Module

**Infosys Springboard Internship 7.0 (Batch 4)**

> _"One platform. Every outlet. Smarter decisions."_

### 📌 Project Overview

This repository delivers the complete, verified, and thoroughly tested implementation of **FranchiseOps AI — Agentic Franchise Operations Intelligence Platform (Milestone 1)**.

The application provides a premium enterprise SaaS authentication interface with dark navy styling, live policy validation, role verification (Franchise Owner, Outlet Manager, Staff / Employee, Administrator), email verification via Gmail SMTP, two-channel password recovery, brute-force lockout protection, role-based admin directory, and JWT session security. Compatible with both local Windows environments and Google Colab.

---

### 🚀 Key Features Implemented

#### 1. Live Password Criteria Checklist & Policy

- **Policy Rules**:
  - Minimum 8 characters.
  - At least 1 uppercase letter (`A-Z`).
  - At least 1 lowercase letter (`a-z`).
  - At least 1 numeric digit (`0-9`).
  - At least 1 special character (`!@#$%^&*()_+-=[]{};':"\\|,.<>/?`~`).
  - Empty and whitespace-only passwords rejected.
- **Live Dynamic Feedback**:
  - Evaluated on every keystroke via `check_password_criteria`.
  - Displays a responsive checklist: neutral grey circle (`○`) before typing, green checkmark (`✓`) when satisfied, and red cross (`✕`) when unsatisfied.
  - Real-time password confirmation matching feedback (`✓ Passwords match` or `✕ Passwords do not match`).
  - Clean form layout with individual policy indicators (no redundant overall success banner).
- **Whitespace Policy**: Passwords with spaces are permitted if complexity criteria are met, and are preserved accurately without silent modification.
- **Unicode & Symbol Safety**: Accented and special characters process safely without encoding errors.
- Shared validation logic across Signup, Security-Question Reset, and OTP Reset.

#### 2. Centered Authentication Card & High-Contrast Design

- **WCAG AAA Readability**: High-contrast crisp white input fields (`#ffffff`) with dark slate typography (`#0f172a`) and purple caret (`#7c3aed`), ensuring 100% legibility across light and dark browser themes.
- **Show/Hide Password Controls**: Native masked inputs with one-click visibility toggle.
- **Segmented Dual Tabs**: Modern pill tabs for **Sign In** and **Sign Up** with mutually exclusive stateful rendering.
- **Role Selection Dropdown**: Sign In includes an enterprise role selector (`Franchise Owner`, `Outlet Manager`, `Staff / Employee`, `Administrator`) with server-side database validation (mismatched roles rejected).
- **Feature Showcase Panel**: Elegant overview cards highlighting platform security capabilities:
  - **Secure Password Hashing** (bcrypt with per-user salt)
  - **JWT Session Security** (HS256 signed access tokens)
  - **Account Lockout Protection** (Brute-force mitigation)
  - **Two-Channel Password Recovery** (Security Questions & Gmail OTP)

#### 3. Email & Username Rules

- **Email Validation**:
  - RFC 5322 compliance with single `@` symbol.
  - 3-Point Validation Rule:
    - $\ge$2 characters before `@` (local part).
    - $\ge$2 characters between `@` and the dot (domain name).
    - $\ge$2 characters after final dot (TLD).
  - Rejection of internal spaces and duplicate `@` symbols.
  - Normalization: Trimmed and lowercased for collision prevention.
- **Username Constraints**:
  - Length between 3 and 50 characters (`[a-zA-Z0-9_.- ]`).
  - Display casing preserved while database lookups enforce case-insensitive uniqueness (`LOWER(username)`).

#### 4. Signup & Database Integrity

- All mandatory fields verified server-side.
- Exact match enforced between Password and Confirm Password.
- Security question selection enforced strictly against an approved fixed list.
- Security answer normalized and hashed using **bcrypt** (never stored in plaintext).
- Rejection of duplicate accounts without creating partial records.
- **Redirect on Registration**: Users are routed to the verification workflow upon successful account creation.

#### 5. Email Verification & Account Activation via Gmail SMTP

- **Automated Verification Code**: Dispatches a cryptographically secure 6-digit activation code via Gmail SMTP upon registration.
- **Login Guard**: Unverified accounts (`is_verified = 0`) are strictly blocked from logging in until email verification succeeds.
- **Security Safeguards**:
  - 10-minute expiration timer.
  - Maximum 3 verification attempts.
  - Single-use enforcement (code marked used immediately upon activation).
  - 60-second cooldown on resending codes.

#### 6. Dual Login & Account Lockout Policy

- Supports sign-in using either **Username** or **Email address**.
- Case-insensitive credential lookup.
- **Lockout Policy**:
  - **Threshold**: 5 consecutive failed login attempts.
  - **Duration**: 15 minutes lockout (`locked_until` timestamp).
  - **Auto-Unlock**: Automatically restores access once the 15-minute window expires.
  - **Tamper Prevention**: Locked accounts cannot authenticate even with the correct password.
- **Anti-Enumeration Defense**: Returns generic error messages (`Invalid username/email or password.`) with cryptographic timing compensation to prevent timing attacks.

#### 7. Separate Admin Authentication & Dashboard

- Admin authentication is maintained separately from ordinary user signup.
- Dedicated `admin_users` table and separate portal login.
- **Admin Dashboard Directory**:
  - Displays registered user IDs, usernames, emails, account status, and registration timestamp.
  - **Data Privacy**: Strictly omits password hashes, security answers, OTPs, and tokens.

#### 8. Password Recovery (Dual Methods)

- **Method 1: Security Question Verification**:
  - Retrieves the registered security question strictly mapped to that account.
  - Compares normalized answer against stored bcrypt hash (cross-account answers rejected).
  - Enforces shared password policy for the new password with live validation.
  - Resets account lockout state upon success and redirects to Login.
- **Method 2: Gmail One-Time Password (OTP)**:
  - Generates secure 6-digit numeric OTPs via Python's cryptographic `secrets` module.
  - Stored as bcrypt hashes with a 5-minute expiration.
  - Enforces single-use policy (marked used immediately).
  - Limits attempts to maximum 3 verification tries.
  - 60-second cooldown on resends.
  - Integrated with Gmail SMTP (SSL on port 465 with TLS port 587 fallback).

#### 9. Responsive 6-Box OTP Email Template

- **Six Individual Horizontal Boxes**: Formatted using an email-client-compatible HTML table (`width: 46px; height: 54px;`) with light lavender backgrounds (`#f3f0ff`), 2px solid indigo borders (`#6366f1`), and large bold dark-indigo digits (`#312e81`).
- **Selectable Continuous Code**: Continuous 6-digit monospace text displayed directly beneath the boxes (`user-select: text`) for effortless manual selection in Gmail.

#### 10. JWT Session Security & Secrets Management

- Issues HS256-signed JWT tokens only upon valid authentication.
- Token signature, role, and expiry validated before rendering dashboard pages.
- Malformed, tampered, or expired tokens immediately trigger session invalidation.
- **Zero Hardcoded Secrets**: Requires explicit configuration in Colab Secrets or environment variables.
- Logout button clears application state and invalidates tokens.

---

### 📂 Project Structure

```text
infosys-franchise-ai/
├── .gitignore                      # Git ignore rules (protects *.db and secrets)
└── Milestone1/
    ├── app.py                      # Main Streamlit web application
    ├── auth_utils.py               # Core authentication, validation, SMTP, & DB utilities
    ├── test_auth.py                # Comprehensive test suite (77 automated tests with pytest)
    ├── requirements.txt            # Python dependencies
    ├── README.md                   # Project documentation and setup guide
    ├── .gitignore                  # Milestone Git ignore rules
    ├── Milestone1.ipynb            # Google Colab submission notebook
    ├── Login_Page.ipynb            # Initial reference notebook
    └── assets/
        └── logo.png                # FranchiseOps AI brain logo
```

---

### 🔐 Google Colab Secrets Configuration

The following Colab Secrets must be created in Google Colab (via the **Key** icon in the left sidebar) with **Notebook access** enabled:

| Secret Name       | Purpose                                     | Value Description                                                                                                                                                |
| :---------------- | :------------------------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `JWT_SECRET`      | Cryptographic JWT signing                   | A secure random string (e.g. 32+ characters)                                                                                                                     |
| `NGROK_AUTHTOKEN` | Ngrok public tunneling                      | Your Ngrok authtoken from your dashboard (dashboard.ngrok.com)                                                                                                   |
| `EMAIL_ADDRESS`   | Gmail sender address for verification & OTP | Your Gmail account (e.g. `yourname@gmail.com`)                                                                                                                   |
| `EMAIL_PASSWORD`  | Gmail App Password for SMTP                 | 16-character Google App Password (from Google Account > Security > 2-Step Verification > App passwords). Note: Any spaces are automatically handled and trimmed. |

> [!NOTE]
> **Google 2-Step Verification Note**: Standard Gmail passwords cannot be used for SMTP. Generate a dedicated 16-character App Password at: `Google Account > Security > 2-Step Verification > App passwords`. The application automatically sanitizes spaces and uses an intelligent dual-port connection (Port 465 SSL with fallback to Port 587 TLS) for maximum reliability in Colab containers.

---

### 🛠️ Local Windows Setup Instructions

1. **Navigate to the Milestone1 directory:**

   ```powershell
   cd d:\infosys-franchise-ai\Milestone1
   ```

2. **Install dependencies:**

   ```powershell
   pip install -r requirements.txt
   ```

3. **Run the automated test suite:**

   ```powershell
   pytest -v test_auth.py
   ```

4. **Launch the Streamlit web application:**
   ```powershell
   streamlit run app.py
   ```
   Open your browser at `http://localhost:8501`.

---
