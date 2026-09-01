# Malware Observation: Windows Defender with "Remus Stealer"

## 📝 Background
This repository documents an independent malware analysis and observation of a malicious payload disguised as legitimate software. 

I downloaded a zip file from a seemingly normal GitHub repository that promised a specific software tool (repository name currently forgotten). Upon extraction, the zip file did not contain standard installation media, but rather a suspicious directory structure: an `installer.exe` executable accompanied by a folder full of random `.dll` files.

Instead of running the file on my host machine, I initiated an investigation. The sequence of events revealed a fascinating gap in Windows Defender's immediate local detection capabilities.

---

## 🔍 Investigation & Timeline

### Phase 1: Sandboxing the Payload (Any.run)
Practicing safe malware handling, my first step was to spin up an environment on [Any.run](https://any.run/) to observe the file's behavior dynamically.

**Findings:**
Upon executing `installer.exe` with default settings, the Any.run sandbox immediately flagged severe malicious activity. 
* The system detected a **Network Trojan**.
* Intrusion Detection Systems (Suricata) flagged the payload specifically as **Remus Stealer**.
* The behavioral analysis showed active attempts to **steal credentials from Web Browsers**.

![Any.run Analysis](https://raw.githubusercontent.com/hamdan-Deb/mal.Observation-Remus/refs/heads/main/imgs/ram_use.png)
*(Behavioral analysis showing network Trojan and browser credential theft)*

### Phase 2: Local AV Failure (The Blind Spot)
Despite the severe malicious nature of the file confirmed by Any.run, my local antivirus—**Windows 11 Security (Microsoft Defender)**—completely failed to detect it at first. 

I ran a customized, targeted scan directly on the folder containing `installer.exe` and the `.dll` files. Even with up-to-date security intelligence, Windows Defender returned a clean bill of health with **"0 threats found."**

![Initial Defender Scan](https://raw.githubusercontent.com/hamdan-Deb/mal.Observation-Remus/refs/heads/main/imgs/no_detect_win11sec.png)
*(Defender scanning 8 files and finding no threats)*

### Phase 3: Cloud Submission to Microsoft
Because the local Defender scan failed, I manually submitted `installer.exe` to the Microsoft Security Intelligence online portal for deep analysis. 

**Findings:**
Once uploaded and processed by Microsoft's cloud analysis, the verdict was clear. Microsoft's backend immediately classified the file as **Malware**, specifically categorizing it as **`Trojan:Win32/Kepavll!rfn`**.

![Microsoft Submission](https://raw.githubusercontent.com/hamdan-Deb/mal.Observation-Remus/refs/heads/main/imgs/ms_submit.png)
*(Microsoft portal classifying the submission as Trojan:Win32/Kepavll!rfn)*

### Phase 4: Local Detection Updates
The most fascinating part of this observation was what happened next. After the cloud submission categorized the threat, the telemetry/signatures updated. 

Only *after* the manual submission to Microsoft did my local Windows Security finally wake up, flag the file as "Severe," and quarantine `installer.exe` from my desktop. 

![Defender Finally Detects](https://raw.githubusercontent.com/hamdan-Deb/mal.Observation-Remus/refs/heads/main/imgs/detect_win11sec.png)
*(Windows Security finally acting on the file post-cloud submission)*

---

## 💡 Key Takeaways & Conclusion

1. **Signatures Lag Behind Obfuscation:** The developers of this malware likely obfuscated or compiled the `.dll` dependencies in a way that successfully evaded Microsoft Defender's local heuristic and static analysis at the time of download.
2. **Cloud vs. Endpoint:** Windows Defender's true power currently lies in its cloud protection. Without cloud communication or someone (like me) actively submitting the sample, an end-user relying strictly on an immediate local right-click scan would have been compromised.
3. **Always Sandbox Unknown Binaries:** The golden rule of cybersecurity proved true here. If I had trusted the "0 threats found" from Windows Defender and run the file locally, my browser credentials would have been scraped and exfiltrated by the Remus Stealer.

## ⚠️ Disclaimer
**This repository is for educational and cybersecurity research purposes only.** Do not execute files from untrusted sources on your host machine. Always use virtual machines, isolated sandboxes, or professional analysis platforms when handling suspicious executables.
