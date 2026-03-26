Task 1 — Classification and Handling of PII Fields

When sharing healthcare data with external researchers, protecting patient privacy is essential. The dataset contains several fields that can directly or indirectly identify individuals. These fields must be handled carefully through removal, masking, or pseudonymization.

Field-wise Classification

1. full_name — Direct PII
Full name directly identifies a person. It has a very high risk of re-identification.
Action: Drop or pseudonymize.
Justification: Names are not required for research analysis. If record linkage is needed, a random patient ID should replace the name.

2. email — Direct PII
Email address is a unique identifier and contact detail.
Action: Drop.
Justification: It provides no analytical value for health research and exposes the individual to privacy risks.

3. date_of_birth — Indirect PII (Quasi-identifier)
Date of birth alone may not identify a person, but combined with other data it can.
Action: Mask by converting to age or age group.
Justification: Age is useful for research while reducing re-identification risk.

4. zip_code — Indirect PII (Quasi-identifier)
ZIP code reveals location and can identify individuals in small areas.
Action: Generalize (e.g., keep only first 3 digits).
Justification: Preserves regional analysis while protecting privacy.

5. job_title — Indirect PII
Some job titles are rare and can identify individuals when combined with other attributes.
Action: Generalize into job category.
Justification: Maintains socioeconomic insight without revealing specific identity.

6. diagnosis_notes — Sensitive Personal Data
Free-text medical notes may contain names, locations, or personal details.
Action: De-identify and clean the text.
Justification: Medical information is highly sensitive and must be stripped of identifying details before sharing.

Example of a Safe Shared Record

Instead of sharing raw data:

Name: Kavitha Saravana Kumar
Email: kavitha@gmail.com

DOB: 15-08-2005
ZIP: 600001
Job: Software Engineer
Notes: Kavitha from Chennai reported chest pain

Share a de-identified version:

Patient_ID: P001
Age_Group: 18–25
Region: 600***
Job_Category: IT Professional
Diagnosis: Chest pain symptoms

Task 2 — Ethical Audit of the API Script
Original Script Issues

The provided script collects patient-level data from a public API and stores it permanently without safeguards. This raises ethical and legal concerns.

Violation 1 — Potential Terms-of-Service and Rate Limit Abuse

The script downloads 100 pages of data using a free-tier API key without checking usage limits or respecting rate restrictions.

Problem:

May exceed allowed usage
Could overload the server
Might violate API terms of service

Corrected Version (Respect Rate Limits):

import requests
import time
import os

API_URL = "https://healthstats-api.example.com/records"
API_KEY = os.getenv("API_KEY")

records = []

for page in range(1, 101):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})

    if response.status_code != 200:
        print("Stopping due to error or limit reached")
        break

    data = response.json()
    records.extend(data["results"])

    time.sleep(1)
Violation 2 — Storing Raw Sensitive Data Permanently

The script saves all collected records, including personally identifiable information, directly into the company database.

Problem:

Violates data minimization principles
Increases risk of data breaches
Unsafe for external sharing

Corrected Version (De-identify Before Storage):

import uuid

def calculate_age(dob):
    from datetime import datetime
    birth_year = int(dob.split("-")[0])
    return datetime.now().year - birth_year

def generalize_job(job_title):
    if "engineer" in job_title.lower():
        return "IT Professional"
    if "doctor" in job_title.lower():
        return "Healthcare Professional"
    return "Other"

def clean_text(notes):
    return notes.replace("Chennai", "").replace("Kavitha", "")

def deidentify(record):
    return {
        "patient_id": str(uuid.uuid4()),
        "age": calculate_age(record["date_of_birth"]),
        "zip_prefix": record["zip_code"][:3],
        "job_category": generalize_job(record["job_title"]),
        "diagnosis_notes": clean_text(record["diagnosis_notes"])
    }

clean_records = [deidentify(r) for r in records]

save_to_database(clean_records)
Conclusion

Before sharing healthcare data externally, it is essential to remove direct identifiers, generalize indirect identifiers, and de-identify sensitive text fields. Additionally, data collection scripts must respect API usage policies and follow ethical data handling practices. These steps help protect patient privacy while still enabling meaningful research.
