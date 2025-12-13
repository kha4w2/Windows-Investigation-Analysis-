

# ✅ **Task 1 — Brute Force Investigation (ELK Stack)**

**Goal:** Identify *when* the brute-force attack happened and extract the *Account Name* and *Workstation Name* involved.

---

## **1. Filtering Logs from the Target Device**

**I opened Kibana Discover and filtered logs using `agent.name` to view only events coming from the targeted machine.**

<img width="975" height="469" alt="image" src="https://github.com/user-attachments/assets/51124abe-c37e-4bef-8de5-b90c88debe3b" />



## **2. Identifying the Failed Logon Event ID**

**I searched Ultimate Windows Security for "Failed to log on" and confirmed the event ID for failed logins is `4625`.**

<img width="975" height="464" alt="image" src="https://github.com/user-attachments/assets/199d06b0-b3d1-4687-b573-3adbba1f368c" />



<img width="975" height="491" alt="image" src="https://github.com/user-attachments/assets/823276d2-4049-4112-9932-e1f6903fcd17" />


## **3. Querying All Failed Login Attempts**

**I searched in Kibana using `event.code: "4625"` to display all failed logon events collected by Winlogbeat.**


<img width="975" height="466" alt="image" src="https://github.com/user-attachments/assets/c3d04c13-4655-4bc4-81ad-fe4b32a9b88d" />



## **4. Detecting Suspicious Login Activity (Brute Force Indicator)**

**From the visualization, I noticed an abnormal spike—130 failed logons on December 6th around 15:00, indicating a potential brute-force attack.**

<img width="975" height="469" alt="image" src="https://github.com/user-attachments/assets/30742437-2c0a-4d97-a23a-d7d19623ae20" />



## **5. Narrowing Down the Exact Attack Time Window**

**I filtered the time range further and identified 82 failed logons within less than two minutes — the exact brute-force activity window (16:33:33 → 16:34:13).**


<img width="975" height="469" alt="image" src="https://github.com/user-attachments/assets/13f7e68a-7c54-4dc1-a183-f5af1053046b" />

<img width="975" height="453" alt="image" src="https://github.com/user-attachments/assets/93fb02b2-320f-4adb-8f41-82c0b2800d38" />


## **6. Extracting Account Name and Workstation Name**

**I opened one of the failed-logon logs and retrieved the required fields:**

* **Account Name:** *Administrator*


<img width="975" height="468" alt="image" src="https://github.com/user-attachments/assets/fd2fe730-c5a9-4286-80ec-1301ccf26b65" />


* **Workstation Name:** *IS-SOC-AR*



<img width="975" height="466" alt="image" src="https://github.com/user-attachments/assets/13c7734f-d370-442b-af10-7a077165fcbd" />


---

# ✅ **Task 2 — Detecting Suspicious `.vbs` File Creation (Sysmon Event ID 11)**

**Goal:** Identify when a `.vbs` file was created and determine the process and path responsible.

---

## **1. Identifying the Relevant Sysmon Event ID**

**I searched Ultimate Windows Security and confirmed that “File Creation” events are logged under Sysmon Event ID `11`.**

<img width="975" height="484" alt="image" src="https://github.com/user-attachments/assets/f17bd4af-493a-4a58-830a-110981a5ab34" />


## **2. Querying All File-Creation Events**

**I filtered Kibana logs using `event.code: "11"` to retrieve all Sysmon file-creation events.**


<img width="975" height="466" alt="image" src="https://github.com/user-attachments/assets/477276b9-2c0f-4e45-a806-4eb291428f16" />



## **3. Narrowing the Scope to `.vbs` Files Only**

**To reduce noise, I filtered further using `file.extension: "vbs"` to return only logs related to VBS file creation.**


<img width="975" height="474" alt="image" src="https://github.com/user-attachments/assets/fe6effcf-a2b8-4de1-9ac9-7243f235d051" />


## **4. Locating the Exact Suspicious File Event**

**Only one result appeared, indicating a `.vbs` file created on December 6th — confirming it as the suspicious file.**


<img width="975" height="472" alt="image" src="https://github.com/user-attachments/assets/67cbd0ab-0aff-475d-9db9-c5465808d1de" />


## **5. Extracting the File Path and Process Details**

**From the event details, I retrieved the file’s full path and the process responsible for creating it:**

* **File Path:**
  `C:\Users\Administrator\Downloads\Iloveturkey\rootdir\x123456.vbs`

* **Creating Process:**
  `C:\Windows\explorer.exe`

* **User:**
  `Administrator`

---

# ✅ **Task 3 — Distinguishing Between Legitimate and Malicious PowerShell Executions**

**Goal:** Identify executed PowerShell scripts and distinguish between legitimate IT activity and malicious execution.

---

## **1. Scoping PowerShell Execution Activity**

**I filtered Kibana logs using the target host and `powershell.exe` to focus only on PowerShell process executions.**


<img width="1917" height="907" alt="image" src="https://github.com/user-attachments/assets/9ec81586-560f-47a1-88aa-e1dddcc9985c" />


## **2. Identifying the Relevant Event Type**

**Since PowerShell execution is a process creation activity, I used Sysmon Event ID `1` to capture all PowerShell executions.**

<img width="1912" height="920" alt="image" src="https://github.com/user-attachments/assets/8cb83e45-d836-486b-a3cc-661c393a9d69" />


## **3. Filtering for PowerShell Script Executions**

**I refined the query further by searching for `.ps1` within the command line to isolate PowerShell script executions.**


<img width="1918" height="922" alt="image" src="https://github.com/user-attachments/assets/c58baab9-09b9-449a-a3b1-18f9c39a741e" />


**Final Query Used:**

```
agent.name: "WIN-LJDLTDHLBH0" AND process.name: "powershell.exe" AND event.code: "1" AND *ps1*
```

---

## **4. Reviewing Retrieved PowerShell Executions**

**The query returned five PowerShell execution events — three legitimate and two suspicious.**

<img width="1918" height="922" alt="image" src="https://github.com/user-attachments/assets/0235911e-e999-43bf-8926-b5c2f771ddee" />


---

## **5. Identifying Legitimate PowerShell Activity**


<img width="1916" height="916" alt="image" src="https://github.com/user-attachments/assets/ed84f8a9-c54b-4c98-9996-79303d16e6c7" />


**Legitimate executions were linked to routine IT operations and matched expected administrative behavior.**

* **Script Name:** `DailyReport.ps1`
* **Script Path:** `C:\IT\Scripts\`
* **Execution Method:** `-File` parameter
* **Parent Process:** `powershell.exe`
* **User:** `Administrator`

---

## **6. Identifying Malicious PowerShell Activity**


<img width="1913" height="922" alt="image" src="https://github.com/user-attachments/assets/b3892902-e20d-481e-980f-9f9735466bd8" />


**Suspicious executions showed indicators of malicious behavior and abnormal execution patterns.**

* **Script Name:** `ts.ps1`
* **Script Path:** `C:\Users\Administrator\Downloads\`
* **Execution Method:** `-Command` with Execution Policy Bypass
* **Parent Process:** `explorer.exe`
* **User:** `Administrator`

---

## **7. Final Classification Summary**

### ✅ Legitimate PowerShell Execution

* Script: **DailyReport.ps1**
* Purpose: Scheduled IT administrative task

### 🚨 Malicious PowerShell Execution

* Script: **ts.ps1**
* Indicators:

  * Executed from user Downloads directory
  * Used execution policy bypass
  * Parented by `explorer.exe`

---

Task4: Identify and Decode Encoded PowerShell Commands
Step 1: Hunt for Encoded PowerShell Execution and Narrow the Scope Using Elastic Query

Searched for PowerShell process creation events containing encoded commands to identify potential obfuscated execution.


<img width="1918" height="905" alt="image" src="https://github.com/user-attachments/assets/e7d852b9-5243-49fe-b4c6-6d3133855071" />


Filtered results using the following query to isolate relevant PowerShell executions:

```
agent.name: WIN-LJDLTDHLBH0 AND process.name: powershell.exe AND event.code: 1 AND process.command_line:*EncodedCommand*
```

<img width="1911" height="912" alt="image" src="https://github.com/user-attachments/assets/e77e3903-6177-494e-99f8-9f7566782991" />


Step 2: Review Process Creation Events

Identified two PowerShell process creation logs where the -EncodedCommand flag was used.

<img width="1907" height="922" alt="image" src="https://github.com/user-attachments/assets/3922b8c3-88a6-4762-b93b-b332099d6121" />

<img width="1910" height="927" alt="image" src="https://github.com/user-attachments/assets/1a8f893d-f0f8-46d4-a017-746aafd00849" />


Step 3: Extract Encoded Commands

Collected the Base64-encoded command strings from the process.command_line field for further analysis.


<img width="1915" height="902" alt="image" src="https://github.com/user-attachments/assets/b9581a0d-f096-4109-91c5-5f74bfc4fb3b" />


<img width="1907" height="917" alt="image" src="https://github.com/user-attachments/assets/9fcc6fd5-6709-4fee-9046-eee071ea9665" />


Step 4: Decode the Encoded Commands

Decoded both Base64 strings using an external decoding tool to reveal the actual executed PowerShell commands.

<img width="1918" height="910" alt="image" src="https://github.com/user-attachments/assets/5ee51e34-074e-45df-ba92-ba7b4c64e985" />

<img width="1918" height="927" alt="image" src="https://github.com/user-attachments/assets/f315c462-261d-4e10-a195-e95344275c37" />


Step 5: Analyze Decoded Output

Reviewed the decoded commands and confirmed they only execute simple echo and Write-Output statements without any malicious behavior.

Final Assessment

Confirmed that only two encoded commands were executed and both are legitimate and non-malicious, indicating no suspicious PowerShell activity.
