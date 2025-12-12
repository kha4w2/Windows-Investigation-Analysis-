

# **Windows Endpoint Compromise Investigation Report**

## **Overview**

In this challenge, I performed a live incident response investigation on a compromised Windows 10 workstation. The system contained an active backdoor installed by **challenge.exe**, and the objective was to identify indicators of compromise, persistence mechanisms, and attacker activity.

First, I installed Sysinternals 

<img width="1059" height="791" alt="Screenshot 2025-12-10 002220" src="https://github.com/user-attachments/assets/bee02f2f-b9fd-4908-b9b8-e1c40a0f706d" />



That is all process in my system 
---

# ✅ **Q1 – Simulate the compromise**

I started by executing `challenge.exe` from an **Administrator Command Prompt** as instructed.

📸 *Insert Screenshot Here*
(*This is the screenshot showing the CMD window running challenge.exe and displaying: “The system has been successfully compromised.”*)

<img width="975" height="664" alt="image" src="https://github.com/user-attachments/assets/b9e66019-60e9-4b7f-b0f7-9f9a1da67590" />


**Answer:** done

---

# ✅ **Q2 – Malware listening port**

Using **Process Explorer** → *challenge.exe → TCP/IP*, I identified the listening port.

<img width="975" height="701" alt="image" src="https://github.com/user-attachments/assets/0c1f4929-1028-4887-be2a-cc0946f496b8" />

(*This screenshot shows TCP 0.0.0.0:50050 → LISTENING*)

<img width="975" height="165" alt="image" src="https://github.com/user-attachments/assets/f0390daf-90b4-4871-9794-51fc4bee805d" />

**Answer:** `50050`

---

# ✅ **Q3 – Process ID of the malware**

In Process Explorer, I located the running instance of **challenge.exe**.

<img width="975" height="662" alt="image" src="https://github.com/user-attachments/assets/17c044a1-a053-44b9-bf47-e76da1bff6b3" />

<img width="975" height="161" alt="image" src="https://github.com/user-attachments/assets/9a9ba4c5-36bb-4b47-9de6-c2b8653a7a53" />


**Answer:** `7328`

---

# ✅ **Q4 – List DLLs loaded by the malware**

From the **DLLs** tab in Process Explorer for PID `7328`, I extracted the loaded modules.
The DLLs beginning with the letter **M** were:

* `msvcrt.dll`
* `mswsock.dll`

<img width="975" height="180" alt="image" src="https://github.com/user-attachments/assets/ce6203bf-7c71-4d15-b5c0-392cec4acef5" />

(*DLLs tab screenshot*)

<img width="975" height="180" alt="image" src="https://github.com/user-attachments/assets/8cb9d5da-f8b6-473c-9a9b-1ab6a98396bc" />

**Answer:** `msvcrt.dll`, `mswsock.dll`

---

# ✅ **Q5 – Malware’s parent process**

Using Process Explorer, I checked the parent tree of `challenge.exe`; As i runed it from cmd
It was spawned by:

<img width="975" height="159" alt="image" src="https://github.com/user-attachments/assets/0078e2f4-da1c-44a0-876f-efe1de1db03f" />


**Answer:** `cmd.exe`

---

# ✅ **Q6 – Shared resources created**

To enumerate system shares, I used:

```
wmic share get name,path
```

I found a suspicious attacker-created share:

**Share Name:** `xkalibur`

<img width="975" height="744" alt="image" src="https://github.com/user-attachments/assets/a665491d-c97b-472d-b33d-e870cb4b9b38" />



<img width="975" height="155" alt="image" src="https://github.com/user-attachments/assets/a7ca33d0-b6a9-4257-9808-6f41830fb7c7" />


**Answer:** `xkalibur`

---

# ✅ **Q7 – Path used by the attacker’s share**

The share mapped to:

```
C:\Users\Khaled\AppData\Local\Temp\46d5b8556d0d3e30ec1
```

<img width="975" height="744" alt="image" src="https://github.com/user-attachments/assets/0ce90e4e-2b1d-4bab-8477-47d8484454cc" />

<img width="975" height="192" alt="image" src="https://github.com/user-attachments/assets/81ef7f7c-e353-43d3-83c5-bb404c6f91fb" />


**Answer:**
`C:\Users\Khaled\AppData\Local\Temp\46d5b8556d0d3e30ec1`

---

# ✅ **Q8 – Persistence mechanism (Registry Run key)**

I launched **Autoruns.exe** from the Sysinternals Suite.
Under **Logon**, I found a suspicious Run entry.

Registry path:

```
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

<img width="975" height="539" alt="image" src="https://github.com/user-attachments/assets/5b2887cf-b7ba-4eae-bc4a-60c3a66386af" />

```

<img width="975" height="277" alt="image" src="https://github.com/user-attachments/assets/258ab8c1-316d-4c02-b685-43181264b96e" />


<img width="975" height="164" alt="image" src="https://github.com/user-attachments/assets/13d33db5-11cb-4b61-8538-789dd5b80d87" />

**Answer:**
`HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`

---

# ✅ **Q9 – Name of the malicious registry entry**

The malicious value was identified as:

<img width="975" height="571" alt="image" src="https://github.com/user-attachments/assets/e5a54f64-ccce-4d1d-95bd-db37d3044303" />


<img width="975" height="172" alt="image" src="https://github.com/user-attachments/assets/13ec839f-de27-4c3b-a45e-08e40e1bfd1d" />

**Answer:** `CleanUpController`

---

# ✅ **Q10 – Full file or image path for the registry entry**

The registry entry points to the following file:

```
C:\Users\tcm\Downloads\wininit.exe
```

<img width="975" height="683" alt="image" src="https://github.com/user-attachments/assets/025883c5-96ed-44d9-8be3-d54a26b64f2a" />

<img width="975" height="195" alt="image" src="https://github.com/user-attachments/assets/5429f120-23cb-4af1-9310-69a6f03aa7fa" />


**Answer:** `C:\Users\tcm\Downloads\wininit.exe`

---

# ✅ **Q11 – Name of the backdoor service installed by the attacker**

The backdoor service is listed under **Services** in Autoruns:

<img width="975" height="185" alt="image" src="https://github.com/user-attachments/assets/78ee4ef4-ad2d-4ae7-b217-f960013e12ad" />

<img width="975" height="185" alt="image" src="https://github.com/user-attachments/assets/5b69ef35-c426-4d76-a0c2-4e813a48a359" />

**Answer:** `WindowsActiveService`


---

# ✅ **Q12 – START_TYPE configuration of the service**

Using the following command:

```
wmic service where "name='WindowsActiveService'" get startmode
```

The service start mode is:

<img width="975" height="174" alt="image" src="https://github.com/user-attachments/assets/03808375-309b-4aba-99c0-209198b6c290" />

**Answer:** `Auto`

<img width="959" height="84" alt="image" src="https://github.com/user-attachments/assets/cc2fbb48-71fc-4580-a3e3-2f915a6348de" />

(command output showing StartMode Auto)

---

# ✅ **Q13 – Full path to the service binary**

The service binary is located at:

```
C:\Users\tcm\Documents\svcbackdoor.exe
```

<img width="975" height="62" alt="image" src="https://github.com/user-attachments/assets/bee1f368-d262-4523-b7c4-8cb54daf126b" />

(command output showing PathName)


<img width="975" height="181" alt="image" src="https://github.com/user-attachments/assets/f123816b-9191-4d69-b266-fd1146858acc" />

**Answer:** `C:\Users\tcm\Documents\svcbackdoor.exe`

---

# ✅ **Q14 – Name of the scheduled task created by the attacker**

Using **Autoruns → Scheduled Tasks**, the attacker created:

<img width="975" height="714" alt="image" src="https://github.com/user-attachments/assets/357e3ad2-f9e9-4f46-8e4c-e45ff89bfae4" />

<img width="975" height="155" alt="image" src="https://github.com/user-attachments/assets/bf426935-de01-471e-a0ea-9843321545ca" />

**Answer:** `/ayttpnzc`

📸 *Insert Screenshot Here (Autoruns Scheduled Tasks tab highlighting /ayttpnzc)*

---

# ✅ **Q15 – Full path to the executable run by the scheduled task**

The scheduled task runs:

```
C:\Users\tcm\Downloads\beacOn.exe
```

<img width="975" height="527" alt="image" src="https://github.com/user-attachments/assets/ec08c25e-6d93-4c29-a736-afdfc0ec0126" />

 (Autoruns showing File Path of scheduled task)

<img width="975" height="173" alt="image" src="https://github.com/user-attachments/assets/41c3a92d-3601-4caa-9cbf-501fb7f409f0" />

**Answer:** `C:\Users\tcm\Downloads\beacOn.exe`

---

# ✅ **Q16 – Scheduled task trigger time**

The scheduled task `/ayttpnzc` is set to run at:


<img width="975" height="159" alt="image" src="https://github.com/user-attachments/assets/2576a0a0-f041-44d3-9115-7fe2ca369444" />


**Answer:** `3:30:00 AM`

<img width="975" height="670" alt="image" src="https://github.com/user-attachments/assets/b547f32d-f0cd-4ccf-b1ad-94b036c97405" />

<img width="975" height="605" alt="image" src="https://github.com/user-attachments/assets/42ae0bc9-de60-4f63-9544-9f0747f7fffe" />

<img width="975" height="715" alt="image" src="https://github.com/user-attachments/assets/54e2872e-0d3e-49d7-987f-3f808d7a2444" />


<img width="975" height="676" alt="image" src="https://github.com/user-attachments/assets/caff7562-3625-4a46-9a51-ddd934b4c3c4" />



 (Autoruns showing task trigger time)

---

# ✅ **Q17 – Clean up and restore the system**

To remediate:

1. Terminate the running `challenge.exe` process (Press **Ctrl + C**).
2. Revert the changes:

```
challenge.exe -revert
```

<img width="975" height="196" alt="image" src="https://github.com/user-attachments/assets/aa74298c-153c-498a-b2f7-71a9f1c7cb67" />

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
