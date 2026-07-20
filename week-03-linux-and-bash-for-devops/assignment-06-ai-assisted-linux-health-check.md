# Assignment 6 — Build an AI-Assisted Linux Health Check (AI-Assisted Linux Incident Triage)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build a read-only Bash triage script that checks the health of your Ubuntu server and Nginx application, connect it to Claude Code as a reusable `/linux-triage` skill, simulate a controlled Nginx incident, use the skill to gather and analyze evidence, recover the service manually, and verify recovery. The workflow follows the Agentic Loop: Gather → Analyze → Human Act → Verify.

---

# Task 1 — Confirm the Healthy Baseline and Create the Workspace

## Goal

Confirm that Nginx and the React application are healthy before building the automation.

### Evidence

#### Screenshot 1 — Output of `systemctl is-active nginx`, `ss -ltn | grep ':80'`, and `curl -I http://localhost`

[Assignment 06Screenshot Task1](screenshots/06-week-03-linux-for-devops-task1-1.png)

---

#### Screenshot 2 — Output of `pwd` and `find . -maxdepth 4 -type d | sort` showing the workspace folder structure

[Assignment 06Screenshot Task1](screenshots/06-week-03-linux-for-devops-task1-2.png)

---

### Notes

Answer the following in your own words:

**1. What proves that Nginx is running?**

When I ran systemctl is-active nginx, it returned active. That tells me the Nginx service is currently running on the server.

---

**2. What proves that the server is listening for HTTP traffic?**

When I ran ss -ltn | grep ':80', I could see a LISTEN entry on port 80. That means the server is open and ready to receive HTTP requests.

---

**3. Why must you capture a healthy baseline before simulating an incident?**

Because I need to know what "normal" looks like before I break anything. Once I simulate the incident, I can compare the failed state to the healthy one to understand exactly what changed. And after I fix it, I can run the checks again to confirm everything is back to how it was.

---

# Task 2 — Create Project Context and Safety Rules in CLAUDE.md

## Goal

Tell Claude exactly what this project does and what it is not allowed to do.

### Evidence

#### Screenshot 3 — CLAUDE.md open in VS Code showing all four sections (Project Overview, Incident Workflow, Safety Rules, Output Rules)

[Assignment 06Screenshot Task2](screenshots/06-week-03-linux-for-devops-task2-3.png)

---

### Notes

Answer the following in your own words:

**1. Why should Claude receive project-specific operational rules?**

Claude needs project-specific rules so it knows exactly what this project is for, which steps to follow during an incident, and which actions it must never perform. Without these rules Claude might make unnecessary changes to the server.

---

**2. Why is the human required to execute the recovery command?**

Because I need to review the evidence first and decide whether the recovery command is actually safe before running it. Claude can recommend it but should never make changes to the server on its own.

---

**3. Which rule prevents Claude from making an unsupported diagnosis?**

The rule "Do not claim a root cause unless the report contains supporting evidence" prevents Claude from guessing a cause that isn't backed by the actual Bash report.

---

# Task 3 — Use Agentic AI to Plan Before Writing the Script

## Goal

Use Claude Code to inspect the environment and produce a read-only plan before creating any Bash code.

### Evidence

#### Screenshot 4 — Claude Code showing the five-check plan and read-only inspection results
[Assignment 06Screenshot Task3](screenshots/06-week-03-linux-for-devops-task2-(1).png)
[Assignment 06Screenshot Task3](screenshots/06-week-03-linux-for-devops-task2-(2).png)


---

### Notes

Answer the following in your own words:

**1. Which part of this task represents the Gather phase?**

The read-only inspection of the Ubuntu server represents the Gather phase. Claude used commands to collect information about Nginx, port 80, the HTTP response, disk usage, and available memory.
---

**2. Did Claude follow the instruction not to create files? How did you verify this?**

Yes, Claude only performed read-only checks and didn't create any files. I verified this by running find . -maxdepth 4 -type f | sort and the only file showing was CLAUDE.md which I created myself.

---

**3. Why is planning before coding useful in DevOps automation?**

Planning helps me decide what the script should check and what each result means before writing any code. It also helps me catch missing or unsafe steps early instead of finding problems after the script is already built.

---

# Task 4 — Build the Linux Triage Bash Script

## Goal

Create one Bash script that gathers consistent Linux and Nginx health evidence.

### Evidence

#### Screenshot 5 — Top section of `linux-triage.sh` showing variables, thresholds, and the checks array

[Assignment 06Screenshot Task4](screenshots/06-week-03-linux-for-devops-task4-5.png)


---

#### Screenshot 6 — Middle section showing check functions and conditionals

[Assignment 06Screenshot Task4](screenshots/06-week-03-linux-for-devops-task4-6.png)
---

#### Screenshot 7 — Bottom section showing the loop, summary function, and exit behavior

[Assignment 06Screenshot Task4](screenshots/06-week-03-linux-for-devops-task4-7.png)

---

#### Screenshot 8 — Output of `bash -n scripts/linux-triage.sh` (no syntax errors) and `ls -l scripts/linux-triage.sh` showing executable permission

[Assignment 06Screenshot Task4](screenshots/06-week-03-linux-for-devops-task4-8.png)
---

### Notes

Answer the following in your own words:

**1. What is stored in the checks array?**
The checks array stores the names of the five functions that perform the health checks: check_service, check_port, check_http, check_disk, and check_memory.
---

**2. How does the `for` loop use that array?**

The for loop reads each function name from the checks array and runs them one at a time in order. This lets the script execute all five health checks consistently.

---

**3. Why are the health checks separated into functions?**

Because each function handles one specific check. This makes the script easier to read, test, and update without affecting the other checks.

---

**4. What is the purpose of `$(...)` in this script?**

$(...) runs a command and captures its output. The script uses it to collect the timestamp, hostname, HTTP status code, disk usage percentage, available memory, and recent Nginx logs.
---

**5. Why does the script use different exit codes for HEALTHY, WARN, and FAIL?**

The exit code shows how serious the issue is. Exit code 0 means all checks passed, 1 means there's a warning, and 2 means at least one check failed. This lets other tools or people quickly understand the result without reading the full report.

---

# Task 5 — Run and Understand the Healthy-State Report

## Goal

Run the Bash script against the healthy server and verify that it creates a report.

### Evidence

####  Screenshot 9 — Output of `./scripts/linux-triage.sh` showing your Full Name and all five check results

[Assignment 06Screenshot Task5](screenshots/06-week-03-linux-for-devops-task5-9.png)

---

#### Screenshot 10 — Output showing the captured exit code and final summary

[Assignment 06Screenshot Task5](screenshots/06-week-03-linux-for-devops-task5-10.png)
---

### Notes

Answer the following in your own words:

**1. What is the overall status of your healthy baseline?**

The overall status is HEALTHY. All five checks passed with no warnings or failures so I can move on to the incident simulation.

---

**2. Which exact Linux evidence proves the application is serving traffic?**

The report shows [PASS] Port 80 is listening and [PASS] Local HTTP check returned status 200. Port 80 listening confirms the server is ready to receive HTTP traffic and the 200 status confirms the React app responded successfully through Nginx.

**3. Did your script return exit code 0 or 1? Explain why.**

The script returned exit code 0 because all five health checks passed. Nginx was active, port 80 was listening, the application returned HTTP 200, and both disk usage at 54% and available memory at 560 MB were well within healthy limits.
---

**4. What is the difference between a warning and a failure in this script?**

A warning means the server is still working but something needs attention, like disk usage being between 80% and 89% or available memory dropping below 100 MB. A failure means a critical check did not pass, like Nginx being inactive, port 80 not listening, the app not returning HTTP 200, or disk usage hitting 90% or higher.
---

# Task 6 — Create and Run the /linux-triage Skill

## Goal

Turn the Bash script into a reusable, manually invoked Agentic AI workflow.

### Evidence

#### Screenshot 11 — `SKILL.md` showing the frontmatter, allowed tool restrictions, and safety rules

[Assignment 06Screenshot Task6](screenshots/06-week-03-linux-for-devops-task6-11.png)

---

#### Screenshot 12 — `/linux-triage` output for the healthy server

[Assignment 06Screenshot Task6](screenshots/06-week-03-linux-for-devops-task6-12.png)

---

### Notes

Answer the following in your own words:

**1. Why does this skill have Bash, Read, and Grep, but not Write?**
The skill needs Bash to run the triage script, Read to open the report file, and Grep to find specific results. It doesn't need Write because Claude should never create or edit files during the triage process.

---

**2. Why is `disable-model-invocation: true` useful for this skill?**

It prevents Claude from running the skill automatically on its own. I have to manually type /linux-triage which keeps me in control of when the server gets inspected.

---

**3. What part is performed by Bash, and what part is performed by Claude?**
Bash runs the script and collects the evidence about Nginx, port 80, HTTP response, disk usage, memory, and recent logs. Claude reads that evidence, explains the results, identifies any failures, and recommends a safe next step without touching the server itself.
---

**4. Why is this better than asking Claude "Is my server healthy?" without giving it evidence?**
Because a general question gives Claude nothing real to work with. The /linux-triage skill first collects actual live evidence from the server using the Bash script, so Claude's answer is based on real data instead of a guess.
---

# Task 7 — Simulate an Nginx Incident and Let the Skill Diagnose It

## Goal

Create a controlled service failure, gather evidence through Bash, and let Claude analyze the evidence without taking recovery action.

### Evidence

#### Screenshot 13 — Output showing Nginx is inactive and the HTTP request fails


[Assignment 06Screenshot Task7](screenshots/06-week-03-linux-for-devops-task7-13.png)


---

#### Screenshot 14 — `/linux-triage` output showing failed evidence, most likely cause, and a suggested recovery command

[Assignment 06Screenshot Task7](screenshots/06-week-03-linux-for-devops-task7-14.png)

---

#### Screenshot 15 — `incident-failure-report.txt` showing the failed checks and your Full Name
[Assignment 06Screenshot Task7](screenshots/06-week-03-linux-for-devops-task7-15.png)
---

### Notes

Answer the following in your own words:

**1. Which three checks failed?**
The Nginx service check, port 80 check, and local HTTP check all failed. The disk and memory checks were not affected by stopping Nginx.
---

**2. What evidence supports the conclusion that Nginx is unavailable?**

The report showed that Nginx is not active, port 80 is not listening, and the local HTTP request returned status 000 meaning connection refused. All three results together confirm that Nginx is down and the application cannot receive any HTTP traffic.

---

**3. Did Claude execute the recovery command? Why is that important?**

No, Claude only recommended systemctl start nginx for my review. This is important because I need to review the evidence and approve the action myself before anything changes on the server. It prevents the AI from making automatic changes during a real incident.
---

**4. Which phase of the Agentic Loop is represented by the Bash report?**

The Bash report represents the Gather phase. The script collected current evidence about Nginx, port 80, the HTTP response, disk usage, memory, and recent logs.

---

**5. Which phase is represented by Claude's explanation?**

Claude's explanation represents the Analyze phase. Claude read the evidence, identified the three failed checks, explained the most likely cause, and recommended a safe recovery command for me to review.

---

# Task 8 — Recover Manually, Verify Again, and Write the Incident Summary

## Goal

Recover the service as the human operator and prove that the system is healthy again.

### Evidence

#### Screenshot 16 — Output showing Nginx is active and `curl -I http://localhost` returns 200 OK

[Assignment 06Screenshot Task8](screenshots/06-week-03-linux-for-devops-task8-16.png)
---

#### Screenshot 17 — Second `/linux-triage` output showing successful recovery with no FAIL results

[Assignment 06Screenshot Task8](screenshots/06-week-03-linux-for-devops-task8-17.png)

---

#### Screenshot 18 — Output of `ls -lah reports` showing both `incident-failure-report.txt` and `recovery-report.txt`

[Assignment 06Screenshot Task8](screenshots/06-week-03-linux-for-devops-task8-18.png)
---

#### Screenshot 19 — `incident-summary.md` showing all required sections and your Full Name

[Assignment 06Screenshot Task8](screenshots/06-week-03-linux-for-devops-task8-19.png)
---

### Notes

Answer the following in your own words:

**1. What action did you execute manually?**

After reviewing the evidence and Claude's recommendation I manually ran sudo systemctl start nginx to bring the Nginx service back up.
---

**2. What evidence proves that the service recovered?**
Running systemctl is-active nginx returned active and curl -I http://localhost returned HTTP/1.1 200 OK. The second /linux-triage run also showed all five checks passing with overall status HEALTHY and no failures.

---

**3. Why is the second triage run necessary?**

Starting Nginx doesn't automatically prove the full application is healthy. The second triage run checks the service, port, HTTP response, disk, and memory again to confirm the entire server returned to a healthy state.

---

**4. What could go wrong if an AI agent automatically restarted every failed service?**

A failed service might have a configuration problem, resource issue, or dependency failure. Automatically restarting it could hide the real problem, cause a restart loop, or make the incident worse. The evidence needs to be reviewed by a human before taking action.

---

**5. In one sentence, explain the difference between using AI as a chatbot and using AI in this agentic workflow.**

A chatbot only answers my question based on what it already knows, but in this agentic workflow Claude used real tools to gather live evidence from the server and based its analysis on actual data while I remained responsible for approving and executing the recovery action.
---

# Incident Summary

Fill in all seven sections below in your own words.

**Full Name:** Spencer Quesie Enchill

**Date:** 17/07/2026

---

**1. Reported Symptom**

The React application was not opening and the local HTTP request could not connect to port 80.

---

**2. Evidence Collected**
The Bash report showed three failed checks: Nginx service is not active, port 80 is not listening, and local HTTP check returned status 000. The recent Nginx logs confirmed the service was stopped and deactivated successfully. Disk usage was 54% and available memory was 371 MB so resource exhaustion was not the cause.

---

**3. Most Likely Cause**
The evidence showed that Nginx had been stopped. Because Nginx was not running port 80 was not listening and the local HTTP request could not reach the server.
---

**4. Human-Approved Recovery Action**
I reviewed Claude's recommendation and manually executed sudo systemctl start nginx.

**5. Verification**

Running systemctl is-active nginx returned active and curl -I http://localhost returned HTTP/1.1 200 OK. The second /linux-triage run confirmed all five checks passed with overall status HEALTHY.

---

**6. Safety Decision**

I allowed the AI skill to gather and analyze evidence but did not allow it to restart Nginx because I needed to review the evidence first and approve the action myself before making any changes to the server.

---

**7. Agentic Loop Mapping**

Gather: The Bash script collected evidence. Analyze: Claude identified the failures and explained the cause. Human Act: I manually started Nginx after reviewing Claude's recommendation. Verify: I ran /linux-triage again and confirmed all five checks passed.
---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`________https://www.linkedin.com/posts/spencerenchill_dmibypravinmishra-agenticai-claudecode-ugcPost-7483847318698614785-LEQu/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAGRQq6IBvhyFikdnuZmUnAgoctctbC0h3m4__________________`

---

#### Screenshot — Published LinkedIn post

[Assignment 06Screenshot Task8](screenshots/06-week-03-linux-for-devops-linkedin.png)


---

# GitHub Repository URL

Paste the URL of your GitHub folder or repository containing the assignment files here:

`Add your URL here`

---

# Submission Instructions

- Add all required screenshots in your submission
- Full Name must be visible in required screenshots and the Bash report
- All written answers must be in your own words
- Do not expose sensitive information (keys, passwords, AWS account IDs, tokens)
- GitHub URL must be included in this document

---

# Completion Checklist

- [ ✅] Task 1: Healthy baseline confirmed, workspace created (Screenshots 1–2, Notes answered)
- [ ✅] Task 2: CLAUDE.md created with all four sections (Screenshot 3, Notes answered)
- [✅ ] Task 3: Five-check plan produced by Claude using read-only tools (Screenshot 4, Notes answered)
- [✅ ] Task 4: `linux-triage.sh` created, syntax validated, executable permission set (Screenshots 5–8, Notes answered)
- [ ✅] Task 5: Healthy-state report generated with no FAIL result (Screenshots 9–10, Notes answered)
- [ ✅] Task 6: `/linux-triage` skill created and run successfully on healthy server (Screenshots 11–12, Notes answered)
- [✅ ] Task 7: Nginx incident simulated, failed evidence captured, Claude did not execute recovery (Screenshots 13–15, Notes answered)
- [ ✅] Task 8: Nginx recovered manually, recovery verified, reports saved, incident summary complete (Screenshots 16–19, Notes answered)
- [✅ ] Incident summary contains all seven required sections
- [✅ ] LinkedIn post published and URL submitted
- [ ✅] Full Name visible in all required screenshots and the Bash report
- [ ✅] Skill does not have Write permission
- [✅] Skill did not execute any recovery commands
- [ ✅] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*