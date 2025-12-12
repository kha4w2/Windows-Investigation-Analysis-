

# **Windows Endpoint Compromise Investigation Report**

## **Overview**

In this challenge, I performed a live incident response investigation on a compromised Windows 10 workstation. The system contained an active backdoor installed by **challenge.exe**, and the objective was to identify indicators of compromise, persistence mechanisms, and attacker activity.

First, I installed Sysinternals 

<img width="1059" height="791" alt="Screenshot 2025-12-10 002220" src="https://github.com/user-attachments/assets/bee02f2f-b9fd-4908-b9b8-e1c40a0f706d" />

<img width="1044" height="735" alt="Screenshot 2025-12-10 002258" src="https://github.com/user-attachments/assets/cc6d5cb9-2483-4332-972c-aeb369110900" />

<img width="1063" height="815" alt="Screenshot 2025-12-10 002318" src="https://github.com/user-attachments/assets/39726f0c-45c8-48ea-bd47-ec1c9a89c1d2" />


That is all process in my system 
---

# ✅ **Q1 – Simulate the compromise**

I started by executing `challenge.exe` from an **Administrator Command Prompt** as instructed.



<img width="990" height="315" alt="Screenshot 2025-12-10 020502" src="https://github.com/user-attachments/assets/5d7f044b-5bec-44d7-9a64-bbe46d81cc7f" />


(*This is the screenshot showing the CMD window running challenge.exe and displaying: “The system has been successfully compromised.”*)

<img width="1339" height="911" alt="Screenshot 2025-12-10 011520" src="https://github.com/user-attachments/assets/daac80b7-d735-416d-8ba5-000c51ce4577" />


**Answer:** done

---

# ✅ **Q2 – Malware listening port**

Using **Process Explorer** → *challenge.exe → TCP/IP*, I identified the listening port.



<img width="1059" height="773" alt="Screenshot 2025-12-10 002459" src="https://github.com/user-attachments/assets/a4fa7b2f-7fc1-444b-a21b-53adb3f1831f" />

<img width="1122" height="819" alt="Screenshot 2025-12-10 002559" src="https://github.com/user-attachments/assets/79571ce1-8dd6-46d6-8ada-b3976e518fbb" />

<img width="1121" height="806" alt="Screenshot 2025-12-10 002758" src="https://github.com/user-attachments/assets/12337d1f-1cae-4330-87c9-69867d10a1de" />

(*This screenshot shows TCP 0.0.0.0:50050 → LISTENING*)


<img width="1641" height="278" alt="Screenshot 2025-12-10 011737" src="https://github.com/user-attachments/assets/97c282f0-b09b-4075-9a16-bd9baa824598" />

**Answer:** `50050`

---

# ✅ **Q3 – Process ID of the malware**

In Process Explorer, I located the running instance of **challenge.exe**.


<img width="1166" height="793" alt="Screenshot 2025-12-10 003201" src="https://github.com/user-attachments/assets/c1dcca35-970f-4857-b4b0-16c957f5bc18" />



**Answer:** `7328`

---

# ✅ **Q4 – List DLLs loaded by the malware**

From the **DLLs** tab in Process Explorer for PID `7328`, I extracted the loaded modules.
The DLLs beginning with the letter **M** were:

* `msvcrt.dll`
* `mswsock.dll`

<img width="1025" height="691" alt="Screenshot 2025-12-10 003806" src="https://github.com/user-attachments/assets/87250d13-3a15-4bac-8051-1acc49abf784" />

(*DLLs tab screenshot*)

<img width="975" height="180" alt="image" src="https://github.com/user-attachments/assets/8cb9d5da-f8b6-473c-9a9b-1ab6a98396bc" />

**Answer:** `msvcrt.dll`, `mswsock.dll`

---

# ✅ **Q5 – Malware’s parent process**

Using Process Explorer, I checked the parent tree of `challenge.exe`; As i runed it from cmd
It was spawned by:

<img width="1328" height="218" alt="Screenshot 2025-12-10 030116" src="https://github.com/user-attachments/assets/ae9a3171-a7f2-4be8-aebd-3a9b796630a7" />


**Answer:** `cmd.exe`

---

# ✅ **Q6 – Shared resources created**

To enumerate system shares, I used:

```
wmic share get name,path
```

<img width="929" height="577" alt="Screenshot 2025-12-10 030307" src="https://github.com/user-attachments/assets/bd8866d4-8eac-4b78-9acf-ac3173d638b5" />

I found a suspicious attacker-created share:

**Share Name:** `xkalibur`


**Answer:** `xkalibur`

---

# ✅ **Q7 – Path used by the attacker’s share**

The share mapped to:

```
C:\Users\Khaled\AppData\Local\Temp\46d5b8556d0d3e30ec1
```

<img width="929" height="577" alt="Screenshot 2025-12-10 030307" src="https://github.com/user-attachments/assets/981921aa-f29d-421b-90af-20aaa1e1e85a" />



**Answer:**
`C:\Users\Khaled\AppData\Local\Temp\46d5b8556d0d3e30ec1`

---

# ✅ **Q8 – Persistence mechanism (Registry Run key)**

I launched **Autoruns.exe** from the Sysinternals Suite.
Under **Logon**, I found a suspicious Run entry.

<img width="891" height="286" alt="Screenshot 2025-12-10 030505" src="https://github.com/user-attachments/assets/82deafc7-defb-4dd7-b9d5-f09fd3416588" />

<img width="923" height="470" alt="Screenshot 2025-12-10 030637" src="https://github.com/user-attachments/assets/fb3a4433-1abb-40b3-a493-633feb3d7e88" />


Registry path:

```
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```


**Answer:**
`HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`

---

# ✅ **Q9 – Name of the malicious registry entry**

The malicious value was identified as:

<img width="920" height="481" alt="Screenshot 2025-12-10 030656" src="https://github.com/user-attachments/assets/a05282ab-6c9d-45d8-a0ea-bc96e9a46bec" />


**Answer:** `CleanUpController`

---

# ✅ **Q10 – Full file or image path for the registry entry**

The registry entry points to the following file:

```
C:\Users\tcm\Downloads\wininit.exe
```

<img width="885" height="597" alt="Screenshot 2025-12-10 030854" src="https://github.com/user-attachments/assets/3fdc25e6-62b9-419f-a671-5f20b585503e" />



**Answer:** `C:\Users\tcm\Downloads\wininit.exe`

---

# ✅ **Q11 – Name of the backdoor service installed by the attacker**

The backdoor service is listed under **Services** in Autoruns:

<img width="927" height="543" alt="Screenshot 2025-12-10 031123" src="https://github.com/user-attachments/assets/ba77d536-d77e-4ffe-b0fc-f5fa76954738" />


**Answer:** `WindowsActiveService`


---

# ✅ **Q12 – START_TYPE configuration of the service**

Using the following command:

```
wmic service where "name='WindowsActiveService'" get startmode
```

The service start mode is:

<img width="1008" height="546" alt="Screenshot 2025-12-10 032028" src="https://github.com/user-attachments/assets/d0bc9597-86ad-4bf1-90a3-1695a70838d6" />

<img width="767" height="67" alt="Screenshot 2025-12-10 032354" src="https://github.com/user-attachments/assets/4b363977-df03-410f-971a-45debc7b2f7e" />

<img width="1304" height="234" alt="Screenshot 2025-12-10 032451" src="https://github.com/user-attachments/assets/59918850-9e3c-42eb-b59f-5c1eafbdfc1d" />

**Answer:** `Auto`

(command output showing StartMode Auto)

---

# ✅ **Q13 – Full path to the service binary**

The service binary is located at:

```
C:\Users\tcm\Documents\svcbackdoor.exe
```

<img width="977" height="63" alt="Screenshot 2025-12-10 032659" src="https://github.com/user-attachments/assets/89d9f110-c013-4eb8-ab93-094eb98f2754" />

(command output showing PathName)


<img width="1334" height="248" alt="Screenshot 2025-12-10 032739" src="https://github.com/user-attachments/assets/6d9fb98a-2a4c-4394-bca5-9c583a9f0607" />

**Answer:** `C:\Users\tcm\Documents\svcbackdoor.exe`

---

# ✅ **Q14 – Name of the scheduled task created by the attacker**

Using **Autoruns → Scheduled Tasks**, the attacker created:

<img width="1053" height="771" alt="Screenshot 2025-12-10 033349" src="https://github.com/user-attachments/assets/6c3812b9-06d9-41a6-9396-f1e548419d2d" />

<img width="1411" height="225" alt="Screenshot 2025-12-10 033413" src="https://github.com/user-attachments/assets/6b8719c9-b25c-4a3e-9014-2448225d317f" />

**Answer:** `/ayttpnzc`

---

# ✅ **Q15 – Full path to the executable run by the scheduled task**

The scheduled task runs:

```
C:\Users\tcm\Downloads\beacOn.exe
```

<img width="1035" height="742" alt="image" src="https://github.com/user-attachments/assets/1586961d-bdbc-4f1a-ae51-b3d3d862a2d2" />

 (Autoruns showing File Path of scheduled task)

<img width="1670" height="297" alt="Screenshot 2025-12-10 033829" src="https://github.com/user-attachments/assets/9331ccd4-facb-4fa3-9e6e-cc55693a08b0" />


**Answer:** `C:\Users\tcm\Downloads\beacOn.exe`

---

# ✅ **Q16 – Scheduled task trigger time**

The scheduled task `/ayttpnzc` is set to run at:


<img width="975" height="159" alt="image" src="https://github.com/user-attachments/assets/2576a0a0-f041-44d3-9115-7fe2ca369444" />


**Answer:** `3:30:00 AM`

<img width="1013" height="547" alt="Screenshot 2025-12-10 033851" src="https://github.com/user-attachments/assets/32783d1c-a6ec-4ccb-9a24-d3a4aeb1b323" />

<img width="1010" height="783" alt="Screenshot 2025-12-10 034218" src="https://github.com/user-attachments/assets/c72f597b-6f34-45bf-bddc-b10367e40142" />

<img width="612" height="526" alt="Screenshot 2025-12-10 034634" src="https://github.com/user-attachments/assets/cd28355b-a886-4bf1-bac1-b34db6a6ee8b" />


<img width="1673" height="272" alt="Screenshot 2025-12-10 034716" src="https://github.com/user-attachments/assets/36344e44-3b71-4147-b0b8-584e57734291" />



 (Autoruns showing task trigger time)

---

# ✅ **Q17 – Clean up and restore the system**

To remediate:

1. Terminate the running `challenge.exe` process (Press **Ctrl + C**).
2. Revert the changes:

```
challenge.exe -revert
```

<img width="1740" height="350" alt="Screenshot 2025-12-10 034753" src="https://github.com/user-attachments/assets/d8e92019-c4be-4989-a4d0-a6cc05a08156" />

(showing cleanup confirmation)*

**Answer:** `done`

---

# ✅ **All Answers Summary (Updated)**

| Question | Answer                                                 |
| -------- | ------------------------------------------------------ |
| Q1       | done                                                   |
| Q2       | 50050                                                  |
| Q3       | 7328                                                   |
| Q4       | msvcrt.dll, mswsock.dll                                |
| Q5       | cmd.exe                                                |
| Q6       | xkalibur                                               |
| Q7       | C:\Users\Khaled\AppData\Local\Temp\46d5b8556d0d3e30ec1 |
| Q8       | HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run     |
| Q9       | CleanUpController                                      |
| Q10      | C:\Users\tcm\Downloads\wininit.exe                     |
| Q11      | WindowsActiveService                                   |
| Q12      | Auto                                                   |
| Q13      | C:\Users\tcm\Documents\svcbackdoor.exe                 |
| Q14      | /ayttpnzc                                              |
| Q15      | C:\Users\tcm\Downloads\beacOn.exe                      |
| Q16      | 3:30:00 AM                                             |
| Q17      | done                                                   |

---
