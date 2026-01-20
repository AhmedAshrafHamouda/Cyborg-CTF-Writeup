# Cyborg – CTF Writeup 🤖

## 📌 Challenge Information
- **Name:** Cyborg  
- **Category:** Web  
- **Platform:** CTF Lab  
- **Difficulty:** Easy  

---

## 📝 Challenge Overview
The challenge exposes a vulnerable web service that allows users to submit a license key.  
The objective is to analyze the backend validation logic and exploit it to retrieve the hidden flag.

---

## 🔍 Initial Recon

### Testing Application Inputs
Initial testing focused on submitting different inputs to understand how the application processes user data.

<img width="491" height="692" alt="Screenshot 2025-10-26 191304" src="https://github.com/user-attachments/assets/5f42c4fd-801d-4cb1-85ee-2ef4a8daf0da" />

---

### Understanding Backend Behavior
By intercepting requests and observing responses, it was found that the application sends user input to:


The request is handled by a backend function named `checkLicense`, which validates the provided license against a secret value stored in an environment variable.

<img width="708" height="477" alt="Screenshot 2025-10-26 191316" src="https://github.com/user-attachments/assets/ac4ad780-cde5-4cc5-ab58-8f3776368d70" />
<img width="1211" height="599" alt="Screenshot 2025-10-26 191547" src="https://github.com/user-attachments/assets/870a28e4-5bcd-4ffe-8b5f-0ae2d38ab8e4" />

---

## 🧠 Vulnerability Analysis

- The backend uses a **regular expression** to compare the submitted license with the flag.
- The flag is stored in an environment variable (e.g. `process.env.FLAG`).
- The comparison uses `!!match`, converting the regex result into a boolean value.

### Key Observation
The `checkLicense` function is asynchronous (`async`), meaning the server waits for the regex evaluation to finish before responding.  
Complex regex patterns introduce noticeable delays, creating a **time-based side channel**.

This behavior is conceptually similar to **Blind Time-Based SQL Injection**, but applied to regex evaluation.

---

## 🧪 Exploitation Strategy – Regex Timing Attack

The goal is to craft **computationally expensive regex patterns** that cause a delay **only when a tested character matches** the flag.

- Matching character → longer response time  
- Non-matching character → normal response time  

To generate suitable patterns:
- AI was used to help generate complex regex payloads
- Each payload was tested and refined using https://regex101.com

<img width="1917" height="325" alt="Screenshot 2025-10-26 191930" src="https://github.com/user-attachments/assets/fbb90981-5c65-4c93-9147-f44e67bdc7a3" />

<img width="1914" height="326" alt="Screenshot 2025-10-26 192003" src="https://github.com/user-attachments/assets/bbde4dd1-d54c-4be8-bec3-ca6c411daa63" />

---

## 🧪 Automated Exploitation

To automate the attack, **Burp Suite Turbo Intruder** was used with a custom script:

- Fuzz the flag character by character
- Measure response time for each request
- Identify the character with the highest response time
- Append the discovered character to a local file (`flag.txt`)
- Repeat until the full flag is recovered

<img width="746" height="523" alt="Screenshot 2025-10-26 192147" src="https://github.com/user-attachments/assets/a542e667-0cae-4216-9457-1bd197e2fce5" />
<img width="930" height="451" alt="Screenshot 2025-10-26 192236" src="https://github.com/user-attachments/assets/febfead1-6038-4ff4-bafa-2e746db8da45" />
<img width="1014" height="451" alt="Screenshot 2025-10-26 192316" src="https://github.com/user-attachments/assets/c17264e1-a35a-48f0-a01e-d04eaedcbab7" />
<img width="1135" height="449" alt="Screenshot 2025-10-26 192440" src="https://github.com/user-attachments/assets/e5f0497a-0ab2-481e-b7f0-fe93e2575077" />

Eventually, random response times were observed, indicating the end of the flag.

<img width="1288" height="451" alt="Screenshot 2025-10-26 192612" src="https://github.com/user-attachments/assets/a5a36111-dea0-4119-bd1a-4d3a9fead15d" />

---

## 🏁 Flag


<img width="1033" height="131" alt="Screenshot 2025-10-26 192518" src="https://github.com/user-attachments/assets/abbd1cfd-1da4-44bf-a5af-8997fe8f06bc" />

---

## 🛠 Tools Used
- Burp Suite (Turbo Intruder)
- regex101
- Browser DevTools

---

## 🛡 Mitigation
- Never compare secrets using regular expressions
- Avoid user-controlled input in regex patterns
- Enforce execution time limits on regex evaluation
- Use constant-time comparison methods

---

## ✍️ Author
**Ahmed Ashraf**  
Cybersecurity | CTF Player | Web Exploitation
