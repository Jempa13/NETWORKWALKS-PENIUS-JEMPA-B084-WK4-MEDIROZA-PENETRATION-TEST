# Security Assessment Report - Mediroza General Hospital

**Repository:** `NETWORKWALKS-PENIUS-JEMPA-B084-WK4-MEDIROZA-PENETRATION-TEST`
**Target:** Mediroza Patient Portal, Pathology PDFs & `mediroza_db_backup_2019.sql` (`mediroza_hr`)
**Date:** 24 November 2026
**Author:** Penius Jempa
**Batch:** B084 - NETWORKWALKS Cybersecurity Internship
**Scope:** Controlled Lab Authorized by Networkwalks (Milestones 1 - 4)
**Overall Risk:** CRITICAL

---

## 1. Executive Summary

This penetration testing report presents the security assessment findings for Mediroza General Hospital and Mediroza CMS 1.4.2 across three main areas: the Patient Portal login interface, three password-protected pathology report PDFs (`S. Dlamini`, `P. Reddy`, and `E. Thompson`), and the database backup file `mediroza_db_backup_2019.sql` (`mediroza_hr` database).

Testing confirmed a critical authentication bypass via SQL injection on the Patient Portal login form, granting access to the authenticated `My lab reports` view without a valid password. Review of `mediroza_db_backup_2019.sql` also exposed sensitive records for **30 staff members** (totaling **ZAR 2,027,000** in monthly payroll, ranging from **ZAR 19,000** to **ZAR 160,000**) and **10 shareholders** in plain text, including national IDs, phone numbers, emails, monthly salaries in ZAR, and shareholdings.

For the three encrypted pathology PDFs, two report passwords (`123456` for `patient_report_1` / Sipho Dlamini and `password` for `patient_report_2` / Priya Reddy) were recovered during testing using the Networkwalks hash calculator and password cracker, while the third report (`E. Thompson`, Lab Ref `LR-2024-1205`) initially resisted dictionary attacks against common leaked passwords (`rockyou.txt`) and the built-in wordlist before being cracked (`!@#$%^&`) using MD5 + RC4 key derivation and symbol/character combination testing.

---

## 2. Scope and Methodology

### Target Scope
- **Patient Portal & Pathology Reports:** Web login portal (`medirozahospital.com/patient/portal.php`), authenticated `My lab reports` dashboard, and three encrypted pathology PDF reports (`S. Dlamini` [`LR-2024-1187`], `P. Reddy` [`LR-2024-1192`], and `E. Thompson` [`LR-2024-1205`]).
- **Database Backup File:** `mediroza_db_backup_2019.sql` (Mediroza CMS 1.4.2 backup module, database `mediroza_hr`, backup header timestamp `2019-08-27 02:14:03`), containing `staff` and `shareholders` tables.

### Tools and Approach
- **Tools Used:** `networkwalks/hash-calculator`, `networkwalks/password-cracker`, John the Ripper (CLI and GUI), `rockyou.txt` wordlist testing, browser login testing, and SQL file inspection.
- **Limitations Encountered:** Hash recognition issues occurred with John the Ripper (`error message "no hash loaded"`). Standard dictionary wordlists (`rockyou.txt` and the built-in 100-entry wordlist) also missed the third PDF password initially because it is a 7-character symbol password (`!@#$%^&`) rather than a common leaked word, requiring MD5 + RC4 key derivation with symbol/character combination testing to recover it.

---

## 3. Findings Summary

| # | Finding | Milestone | Risk Rating | Status |
| :--- | :--- | :--- | :--- | :--- |
| 1 | SQL Injection Authentication Bypass (Patient Portal Login) | M1 | CRITICAL | Open |
| 2 | Exposure of Confidential HR & Shareholder Database Backup (`mediroza_db_backup_2019.sql`) | M3 | CRITICAL | Open |
| 3 | Recoverable & Legacy Encryption Passwords on Pathology PDFs (Including Third PDF) | M2 | HIGH | Open |
| 4 | Database Backup Date / Metadata Anomalies | M3 | MEDIUM | Open |

---

## 4. Detailed Findings and Proof of Exploitation

### Finding 1: SQL Injection Authentication Bypass (Patient Portal Login)

Entering `admin'--` in the Username field with a placeholder password (such as `test`) bypassed authentication and logged into the Patient Portal (`medirozahospital.com/patient/portal.php`), displaying the `My lab reports` page with downloadable encrypted pathology PDFs for `S. Dlamini` (`LR-2024-1187`), `P. Reddy` (`LR-2024-1192`), and `E. Thompson` (`LR-2024-1205`). This occurs when user input is concatenated directly into a SQL query, allowing `--` to comment out the remaining password condition.

**Screenshot Placeholders:**
- `Evidence/M1_SQLi_Authentication_Bypass/01_patient_portal_login.png`
- `Evidence/M1_SQLi_Authentication_Bypass/02_my_lab_reports_dashboard.png`

---

### Finding 2: Exposure of Confidential HR and Shareholder Records in SQL Backup

Inspecting `mediroza_db_backup_2019.sql` exposed unencrypted `INSERT` statements for **30 staff members** and **10 shareholders**:

- **Staff Table (30 records):** Includes `full_name`, `job_title`, `department`, `email`, `phone`, `national_id`, `monthly_salary_zar` (`ZAR 19,000` to `ZAR 160,000`, totaling `ZAR 2,027,000` monthly payroll), and `date_joined`.
- **Shareholders Table (10 records):** Includes `shareholder_name`, `share_percent`, `shares_held` (`1,000,000` total shares), and `share_class` (`Ordinary` and `Preferential`).
- **Data Anomalies:** Several staff records list `date_joined` values in late 2019 and early 2020 (such as `2019-09-02`, `2019-11-05`, and `2020-02-10`) despite the file's `2019-08-27` backup header.

**Screenshot Placeholders:**
- `Evidence/M3_Database_Backup_Exposure/01_mediroza_sql_backup_staff_dump.png`
- `Evidence/M3_Database_Backup_Exposure/02_mediroza_sql_backup_shareholders_dump.png`

#### Shareholder Records (`mediroza_hr.shareholders`)

| ID | Shareholder Name | Share % | Shares Held | Share Class |
| :---: | :--- | :---: | :---: | :--- |
| 1 | Dr. Rajesh Naidoo | 18.00% | 180,000 | Ordinary |
| 2 | Cedar Health Holdings (Pty) Ltd | 15.00% | 150,000 | Ordinary |
| 3 | Dr. Johan van der Merwe | 12.00% | 120,000 | Ordinary |
| 4 | Reddy Family Trust | 11.00% | 110,000 | Ordinary |
| 5 | Thabo Molefe | 10.00% | 100,000 | Ordinary |
| 6 | Sarah Botha | 9.00% | 90,000 | Ordinary |
| 7 | Dr. Ahmed Kara | 8.00% | 80,000 | Preferential |
| 8 | Naledi Zulu | 7.00% | 70,000 | Ordinary |
| 9 | Michael Roberts | 6.00% | 60,000 | Ordinary |
| 10 | Dr. Vikram Chetty | 4.00% | 40,000 | Preferential |

#### Staff Records (`mediroza_hr.staff` — Total Employees: 30 | Total Monthly Payroll: ZAR 2,027,000)

| ID | Full Name | Job Title | Department | Email | Phone | National ID | Salary (ZAR) | Date Joined |
| :---: | :--- | :--- | :--- | :--- | :--- | :---: | :---: | :---: |
| 1 | Dr. Rajesh Naidoo | Chief Pathologist | Diagnostics Lab | r.naidoo@medirozahospital.com | +27 82 101 2007 | 85021013011081 | 138,000 | 2009-03-16 |
| 2 | Sarah Botha | Chief Financial Officer | Finance | s.botha@medirozahospital.com | +27 82 102 2014 | 85031023022082 | 152,000 | 2011-07-01 |
| 3 | Dr. Johan van der Merwe | Medical Director | Management | j.merwe@medirozahospital.com | +27 82 103 2021 | 85041033033083 | 160,000 | 2007-01-22 |
| 4 | Dr. Anita Naicker | Consultant Cardiologist | Cardiology | a.naicker@medirozahospital.com | +27 82 104 2028 | 85051043044084 | 132,000 | 2012-09-10 |
| 5 | Dr. Ahmed Kara | Consultant Physician | Internal Medicine | a.kara@medirozahospital.com | +27 82 105 2035 | 85061053055085 | 128,000 | 2013-02-18 |
| 6 | Dr. Yusuf Cassim | Senior Registrar | Emergency & Trauma | y.cassim@medirozahospital.com | +27 82 106 2042 | 85071063066086 | 74,000 | 2018-05-04 |
| 7 | Michael Roberts | HR Director | Human Resources | m.roberts@medirozahospital.com | +27 82 107 2049 | 85081073077087 | 96,000 | 2010-11-15 |
| 8 | Susan Pretorius | HR Officer | Human Resources | s.pretorius@medirozahospital.com | +27 82 108 2056 | 85091083088088 | 32,000 | 2016-08-23 |
| 9 | Jameel Malik | IT Systems Administrator | IT | j.malik@medirozahospital.com | +27 82 109 2063 | 85101093099080 | 58,000 | 2015-04-12 |
| 10 | Thabo Molefe | Network Engineer | IT | t.molefe@medirozahospital.com | +27 82 110 2070 | 85111103110081 | 46,000 | 2017-10-02 |
| 11 | Nomvula Khumalo | Registered Nurse | Emergency & Trauma | n.khumalo@medirozahospital.com | +27 82 111 2077 | 85121113121082 | 34,000 | 2016-01-19 |
| 12 | Lerato Mokoena | Registered Nurse | Pediatrics | l.mokoena@medirozahospital.com | +27 82 112 2084 | 85011123132083 | 33,000 | 2017-06-07 |
| 13 | Bongani Ndlovu | Registered Nurse | Cardiology | b.ndlovu@medirozahospital.com | +27 82 113 2091 | 85021133143084 | 35,000 | 2015-12-01 |
| 14 | Zanele Mahlangu | Nursing Sister | Theatre | z.mahlangu@medirozahospital.com | +27 82 114 2098 | 85031143154085 | 42,000 | 2013-03-25 |
| 15 | Kagiso Sithole | Pharmacist | Pharmacy | k.sithole@medirozahospital.com | +27 82 115 2105 | 85041153165086 | 61,000 | 2014-09-08 |
| 16 | Naledi Zulu | Pharmacy Assistant | Pharmacy | n.zulu@medirozahospital.com | +27 82 116 2112 | 85051163176087 | 26,000 | 2019-02-14 |
| 17 | Themba Nkosi | Radiographer | Radiology | t.nkosi@medirozahospital.com | +27 82 117 2119 | 85061173187088 | 44,000 | 2016-07-30 |
| 18 | Palesa Radebe | Radiographer | Radiology | p.radebe@medirozahospital.com | +27 82 118 2126 | 85071183198080 | 43,000 | 2017-04-11 |
| 19 | Deepak Pillay | Lab Technologist | Diagnostics Lab | d.pillay@medirozahospital.com | +27 82 119 2133 | 85081193209081 | 41,000 | 2015-05-20 |
| 20 | Kavitha Govender | Lab Technician | Diagnostics Lab | k.govender@medirozahospital.com | +27 82 120 2140 | 85091203220082 | 35,000 | 2018-08-06 |
| 21 | Dr. Suresh Moodley | Consultant Radiologist | Radiology | s.moodley@medirozahospital.com | +27 82 121 2147 | 85101213231083 | 130,000 | 2012-02-28 |
| 22 | Dr. Fatima Patel | Pediatrician | Pediatrics | f.patel@medirozahospital.com | +27 82 122 2154 | 85111223242084 | 118,000 | 2013-10-17 |
| 23 | Nisha Singh | Physiotherapist | Rehabilitation | n.singh@medirozahospital.com | +27 82 123 2161 | 85121233253085 | 48,000 | 2016-11-09 |
| 24 | Dr. Vikram Chetty | Anaesthetist | Theatre | v.chetty@medirozahospital.com | +27 82 124 2168 | 85011243264086 | 135,000 | 2011-06-13 |
| 25 | David Smith | Facilities Manager | Operations | d.smith@medirozahospital.com | +27 82 125 2175 | 85021253275087 | 52,000 | 2014-01-27 |
| 26 | Karen O'Connor | Billing Administrator | Finance | k.oconnor@medirozahospital.com | +27 82 126 2182 | 85031263286088 | 29,000 | 2018-03-19 |
| 27 | James Wilson | Security Supervisor | Operations | j.wilson@medirozahospital.com | +27 82 127 2189 | 85041273297080 | 27,000 | 2019-09-02 |
| 28 | Linda Fourie | Receptionist | Front Office | l.fourie@medirozahospital.com | +27 82 128 2196 | 85051283308081 | 19,000 | 2020-02-10 |
| 29 | Peter van Wyk | Procurement Officer | Supply Chain | p.wyk@medirozahospital.com | +27 82 129 2203 | 85061293319082 | 38,000 | 2015-08-24 |
| 30 | Andile Mbeki | Ward Clerk | Administration | a.mbeki@medirozahospital.com | +27 82 130 2210 | 85071303330083 | 21,000 | 2019-11-05 |

---

### Finding 3: Encrypted Pathology PDF Password Cracking (Including Third PDF)

All three patient pathology report PDFs were unlocked during testing using the Networkwalks hash calculator and password cracker:

- **First PDF (`patient_report_1` — Sipho Dlamini, Patient ID `MG-P-10231`, Lab Ref `LR-2024-1187`):** Cracked with password `123456`.
- **Second PDF (`patient_report_2` — Priya Reddy, Patient ID `MG-P-10244`, Lab Ref `LR-2024-1192`):** Cracked with password `password`.
- **Third PDF (`patient_report_3` — Emily Thompson, Patient ID `MG-P-10255`, DOB `1978-09-03`, Lab Ref `LR-2024-1205`, Referring Dr. Ahmed Kara):** Initially resisted `rockyou.txt` and the built-in wordlist because its password (`!@#$%^&`) consists strictly of 7 special symbols rather than dictionary words. The PDF uses Revision 3 (128-bit) encryption, which derives keys through MD5 + RC4; testing symbol/character combinations (`!@#$%^&`) matched the hash and unlocked Emily Thompson's report.

**Screenshot Placeholders:**
- `Evidence/M2_PDF_Hash_Extraction_and_Cracking/01_patient_report_1_cracked_and_report.png`
- `Evidence/M2_PDF_Hash_Extraction_and_Cracking/02_patient_report_2_cracked_and_report.png`
- `Evidence/M2_PDF_Hash_Extraction_and_Cracking/03_patient_report_3_cracked_and_report.png`

---

## 6. Risk Rating

| Finding | Risk Rating | Justification |
| :--- | :--- | :--- |
| SQL Injection Authentication Bypass (Patient Portal) | CRITICAL | Allows bypassing login authentication with `admin'--` without knowing a password, exposing access to patient lab report records. |
| Exposed Database Backup (`mediroza_db_backup_2019.sql`) | CRITICAL | Exposes national IDs, contact details, salaries, and shareholding records for 30 staff and 10 shareholders in plain text. |
| Pathology PDF Password Recovery (3 Reports) | HIGH | All three pathology PDF passwords (`123456`, `password`, and `!@#$%^&`) were recovered, exposing sensitive medical reports; although `!@#$%^&` avoided standard dictionary words, legacy R3 (128-bit MD5 + RC4) and symbol combinations remained crackable. |
| Database Backup Date / Metadata Anomalies | MEDIUM | Post-2019 join dates inside a 2019-stamped backup point to data hygiene or backup versioning issues. |

---

## 7. Recommendations and Remediation

- **Use Parameterized Queries & Secure Password Hashing:** Replace dynamic SQL string concatenation on the login form with prepared statements, and verify hashed passwords (such as `bcrypt` or `Argon2`) in application code.
- **Remove Exposed Backups & Restrict Access:** Remove `mediroza_db_backup_2019.sql` from public or unauthorized locations, store backups securely with access controls and encryption, and rotate exposed credentials.
- **Upgrade Document Encryption & Access Controls:** Move away from weak/common passwords (`123456`, `password`) and predictable symbol sequences (`!@#$%^&`), upgrade PDF encryption beyond legacy R3 (MD5/RC4), and enforce proper per-user access controls and backup audit hygiene.
```[cite: 13, 14, 15, 16, 17, 18, 19, 20]
