# Cyborg – CTF Writeup 🤖

## 📌 Challenge Information
- **Name:** Cyborg  
- **Category:** Web  
- **Platform:** CTF Lab  
- **Difficulty:** Easy  

---

## 📝 Challenge Overview
The challenge provides a vulnerable web service that validates a user-supplied license key.
The objective is to analyze the backend logic using the provided source code and exploit it to retrieve the hidden flag.

---

## 🔍 Initial Recon

### Testing Application Inputs
Initial testing involved submitting different inputs to observe how the application behaves and how responses change based on the supplied license.

<img width="491" height="692" alt="Screenshot 2025-10-26 191304" src="https://github.com/user-attachments/assets/5f42c4fd-801d-4cb1-85ee-2ef4a8daf0da" />

---

### Understanding Backend Logic (Source Code Analysis)
Since the source code is provided with the challenge, the main focus was on reviewing it to understand how license validation is implemented.

The application sends user input to the following endpoint: ** POST /license


The request is processed by a backend function named `checkLicense`.

<img width="708" height="477" alt="Screenshot 2025-10-26 191316" src="https://github.com/user-attachments/assets/ac4ad780-cde5-4cc5-ab58-8f3776368d70" />
<img width="1211" height="599" alt="Screenshot 2025-10-26 191547" src="https://github.com/user-attachments/assets/870a28e4-5bcd-4ffe-8b5f-0ae2d38ab8e4" />

---

## 🧠 Vulnerability Analysis

From reviewing the source code, the following behavior was identified:

- The `checkLicense` function validates the provided license using a **regular expression**.
- The regex is compared against a secret value.
- In the provided source code, this value appears as an environment variable **only for local testing purposes**.
- In the actual lab environment, the secret is stored **either in a database or locally on the server**, not necessarily as an environment variable.
- The result of the regex match is converted to a boolean using `!!match`.

### Key Observation
The `checkLicense` function is declared as `async`, meaning the server waits for the regex evaluation to complete before sending a response.

Complex regex patterns introduce measurable delays in processing time.  
This creates a **time-based side channel**, similar in concept to **Blind Time-Based SQL Injection**, but applied to regex evaluation.

---

## 🧪 Exploitation Strategy – Regex Timing Attack

The attack relies on crafting **computationally expensive regex patterns** that cause a delay **only when a guessed character matches the secret**.

- Matching character → increased response time  
- Non-matching character → normal response time  

To achieve this:
- Complex regex patterns were generated and refined
- Patterns were tested and analyzed using https://regex101.com to confirm timing behavior

<img width="1917" height="325" alt="Screenshot 2025-10-26 191930" src="https://github.com/user-attachments/assets/fbb90981-5c65-4c93-9147-f44e67bdc7a3" />

<img width="1914" height="326" alt="Screenshot 2025-10-26 192003" src="https://github.com/user-attachments/assets/bbde4dd1-d54c-4be8-bec3-ca6c411daa63" />

---

## 🧪 Automated Exploitation

To automate the extraction process, **I wrote a custom Turbo Intruder script** in Burp Suite:

- The script fuzzes the secret **character by character**
- Measures the response time for each request
- The character that produces the highest response time is considered correct
- Each discovered character is appended to a local file (`flag.txt`)
- The process continues until the full flag is recovered

<img width="746" height="523" alt="Screenshot 2025-10-26 192147" src="https://github.com/user-attachments/assets/a542e667-0cae-4216-9457-1bd197e2fce5" />
<img width="930" height="451" alt="Screenshot 2025-10-26 192236" src="https://github.com/user-attachments/assets/febfead1-6038-4ff4-bafa-2e746db8da45" />
<img width="1014" height="451" alt="Screenshot 2025-10-26 192316" src="https://github.com/user-attachments/assets/c17264e1-a35a-48f0-a01e-d04eaedcbab7" />
<img width="1135" height="449" alt="Screenshot 2025-10-26 192440" src="https://github.com/user-attachments/assets/e5f0497a-0ab2-481e-b7f0-fe93e2575077" />

Once random characters and inconsistent timings were observed, it indicated that the **end of the flag** had been reached.

<img width="1288" height="451" alt="Screenshot 2025-10-26 192612" src="https://github.com/user-attachments/assets/a5a36111-dea0-4119-bd1a-4d3a9fead15d" />

---

## 🏁 Flag

FLAG{
<img width="1033" height="131" alt="Screenshot 2025-10-26 192518" src="https://github.com/user-attachments/assets/abbd1cfd-1da4-44bf-a5af-8997fe8f06bc" />
}

---

## 🛠 Tools Used
- Burp Suite (Turbo Intruder – custom script)
- regex101
- VS Code
