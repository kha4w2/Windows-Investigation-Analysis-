

# ✅ **Task 1 — Brute Force Investigation (ELK Stack)**

**Goal:** Identify *when* the brute-force attack happened and extract the *Account Name* and *Workstation Name* involved.

---

## **1. Filtering Logs from the Target Device**

**I opened Kibana Discover and filtered logs using `agent.name` to view only events coming from the targeted machine.**

---

## **2. Identifying the Failed Logon Event ID**

**I searched Ultimate Windows Security for "Failed to log on" and confirmed the event ID for failed logins is `4625`.**

---

## **3. Querying All Failed Login Attempts**

**I searched in Kibana using `event.code: "4625"` to display all failed logon events collected by Winlogbeat.**

---

## **4. Detecting Suspicious Login Activity (Brute Force Indicator)**

**From the visualization, I noticed an abnormal spike—130 failed logons on December 6th around 15:00, indicating a potential brute-force attack.**

---

## **5. Narrowing Down the Exact Attack Time Window**

**I filtered the time range further and identified 82 failed logons within less than two minutes — the exact brute-force activity window (16:33:33 → 16:34:13).**

---

## **6. Extracting Account Name and Workstation Name**

**I opened one of the failed-logon logs and retrieved the required fields:**

* **Account Name:** *Administrator*
* **Workstation Name:** *IS-SOC-AR*

---

# ✅ **Task 2 — Detecting Suspicious `.vbs` File Creation (Sysmon Event ID 11)**

**Goal:** Identify when a `.vbs` file was created and determine the process and path responsible.

---

## **1. Identifying the Relevant Sysmon Event ID**

**I searched Ultimate Windows Security and confirmed that “File Creation” events are logged under Sysmon Event ID `11`.**

---

## **2. Querying All File-Creation Events**

**I filtered Kibana logs using `event.code: "11"` to retrieve all Sysmon file-creation events.**

---

## **3. Narrowing the Scope to `.vbs` Files Only**

**To reduce noise, I filtered further using `file.extension: "vbs"` to return only logs related to VBS file creation.**

---

## **4. Locating the Exact Suspicious File Event**

**Only one result appeared, indicating a `.vbs` file created on December 6th — confirming it as the suspicious file.**

---

## **5. Extracting the File Path and Process Details**

**From the event details, I retrieved the file’s full path and the process responsible for creating it:**

* **File Path:**
  `C:\Users\Administrator\Downloads\Iloveturkey\rootdir\x123456.vbs`

* **Creating Process:**
  `C:\Windows\explorer.exe`

* **User:**
  `Administrator`

---


