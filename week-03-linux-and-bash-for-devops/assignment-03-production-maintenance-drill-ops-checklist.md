# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI


[Assignment 03Screenshot 1](screenshots/03-week-03-linux-for-devops.png)
---

#### Screenshot 2 — Output of `ip a`


[Assignment 03Screenshot 2](screenshots/03-week-03-linux-for-devops2.png)
---

#### Screenshot 3 — Output of `sudo ss -tulpen`

[Assignment 03Screenshot 2](screenshots/03-week-03-linux-for-devops3.png)

---

#### Screenshot 4 — Output of `sudo ufw status`

[Assignment 03Screenshot 2](screenshots/03-week-03-linux-for-devops4.png)
ufw is inactive because firewall rules are being managed at the AWS Security Group level instead. Port 22 and port 80 are controlled there.
---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

I ran sudo ss -tulpen and saw a TCP LISTEN entry on 0.0.0.0:80 with the process name nginx , this confirms Nginx is actively accepting HTTP connections on all network interfaces.

---

**2. What proves SSH is active on port 22?**

I can see sshd listed as a LISTEN process on 0.0.0.0:22 in the ss output and the fact that I'm actively connected to this server right now via SSH further confirms it's working.

---

**3. Did you find any unexpected open ports? Explain briefly.**

No unexpected ports. The only externally relevant ports were 22 (SSH) and 80 (Nginx), both of which are expected. The other entries (ports 53 and 323) were bound to 127.0.0.x — localhost only meaning they're internal to the server and not reachable from outside.

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

[Assignment 03Screenshot Task2-1](screenshots/03-week-03-linux-for-devops-task2-1.png)


---

#### Screenshot 2 — Output of `sudo nginx -t`
[Assignment 03Screenshot Task2-2](screenshots/03-week-03-linux-for-devops-task2-2.png)

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

[Assignment 03Screenshot Task2-3](screenshots/03-week-03-linux-for-devops-task2-3.png)

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

If Nginx fails to restart, the web server goes down and users get a connection error when trying to reach the site. No HTTP traffic gets served the application becomes completely unavailable until Nginx is recovered. That's why I always run sudo nginx -t before restarting, to catch config errors before they cause downtime.
---

**2. What's your basic rollback plan?**

If a config change breaks Nginx, I would revert the change in /etc/nginx/sites-available/default using sudo nano, run sudo nginx -t to confirm the syntax is clean again, then run sudo systemctl restart nginx to bring it back up. If I wasn't sure what changed, I could also use git diff to compare against a known good version of the config file.
---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

[Assignment 03Screenshot Task3-1](screenshots/03-week-03-linux-for-devops-task3-1.png)
---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

[Assignment 03Screenshot Task3-2](screenshots/03-week-03-linux-for-devops-task3-2.png)
---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`


[Assignment 03Screenshot Task3-3](screenshots/03-week-03-linux-for-devops-task3-3.png)

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

No errors were found in either log. The error log returned no output at all, and the journalctl entries show only clean Started, Stopped, and Deactivated successfully events no failed or crashed entries anywhere.

---

**2. If there were no errors, what does that indicate about the system?**

An empty error log and clean journalctl history means Nginx hasn't encountered any internal errors or failed lifecycle events during this period. It's a positive signal about current system health, though it only reflects the window checked  logs should be reviewed regularly as traffic and usage grow.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes. When I ran curl http://100.58.92.202 earlier, it appeared in the access log as a GET request returning a 200 status with the curl user agent. This confirms the full traffic path is working end-to-end  the request left the client, reached Nginx, was served correctly, and was logged.
---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`
[Assignment 03Screenshot Task4-1](screenshots/03-week-03-linux-for-devops-task4-1.png)

---

#### Screenshot 2 — Output of `free -h`
[Assignment 03Screenshot Task4-2](screenshots/03-week-03-linux-for-devops-task4-2.png)
---

#### Screenshot 3 — Output of `df -h`

[Assignment 03Screenshot Task4-3](screenshots/03-week-03-linux-for-devops-task4-3.png)

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

[Assignment 03Screenshot Task4-4](screenshots/03-week-03-linux-for-devops-task4-4.png)

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

I ran uptime, free -h, df -h, and sudo du -sh /var/* | sort -h to check CPU, memory, and disk. None of the three show any critical signal right now  CPU load is 0.00, memory has 555Mi available, and disk is at 50%. If I had to flag one for ongoing attention it would be disk, since /var/lib (370M) and /var/cache (150M) can grow quietly over time through package installs and log accumulation, unlike CPU or memory pressure which usually show visible symptoms first.
---

**2. What happens if disk becomes 100% full in a production server?**
Logs stop writing new entries, which is dangerous because that's exactly when you need them most during an active incident. Applications fail if they need temporary disk space to operate. Package managers and build tools break. If a database were running locally, it could refuse writes or become corrupted. In severe cases even SSH access can become unreliable, making recovery much harder.

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

[Assignment 03Screenshot Task5-1](screenshots/03-week-03-linux-for-devops-task5-1.png)


---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

[Assignment 03Screenshot Task5-2](screenshots/03-week-03-linux-for-devops-task5-2.png)

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

[Assignment 03Screenshot Task5-3](screenshots/03-week-03-linux-for-devops-task5-3.png)


---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

I confirmed the correct version through three checks. First, ls -lah /var/www/html showed a genuine React production build in place — index.html, a static/ folder with compiled JS/CSS bundles, and standard CRA metadata files like asset-manifest.json and manifest.json. Second, grep -R "Deployed by" found my personalization text compiled into the live JavaScript bundle inside /var/www/html, proving this specific build not a generic or stale one — is what's actually being served. Third, grep -n "try_files" confirmed Nginx is correctly configured to fall back to index.html for unmatched routes, ensuring the React SPA works correctly beyond just the homepage.


---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

[Assignment 03Screenshot Task6-1](screenshots/03-week-03-linux-for-devops-task6-1.png)

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

[Assignment 03Screenshot Task6-2](screenshots/03-week-03-linux-for-devops-task6-2.png)

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

[Assignment 03Screenshot Task6-3](screenshots/03-week-03-linux-for-devops-task6-3.png)

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

I removed the semicolon from the try_files $uri $uri/ /index.html; directive on line 51 of /etc/nginx/sites-available/default. Without the semicolon, Nginx's parser couldn't tell where that directive ended, so it kept reading into the next line and reported an unexpected } error one line later than the actual mistake.

---

**2. How did you fix the issue?**

I reopened the config file with sudo nano, navigated back to line 51, and restored the missing semicolon. I then ran sudo nginx -t to confirm the syntax was valid before touching the live service. Only after seeing syntax is ok did I run sudo systemctl restart nginx, followed by curl -I to confirm the app was serving correctly with a 200 OK response.


---

**3. How can you avoid this kind of issue in real production systems?**

Always run sudo nginx -t after any config change before restarting  this catches syntax errors without risking downtime. Keep Nginx config files in version control so any bad change can be instantly reverted. Use a staging environment to test config changes before they reach production. And where possible, automate config validation in a CI/CD pipeline so broken configs never reach the live server at all.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

[Assignment 03Screenshot Task7-1](screenshots/03-week-03-linux-for-devops-task7-1.png)

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

[Assignment 03Screenshot Task7-2](screenshots/03-week-03-linux-for-devops-task7-2.png)

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

I moved the entire /var/www/html directory to a backup location and replaced it with an empty directory at the same path. Nginx remained running and correctly configured, but with no content in the web root  not even an index.html fallback it returned a 403 Forbidden error instead of serving the React app.

---

**2. How did you fix the issue and restore the application?**
I deleted the empty broken directory with sudo rm -rf /var/www/html, then restored the original deployment by moving the backup back into place with sudo mv /var/www/html_backup /var/www/html. After restarting Nginx with sudo systemctl restart nginx, I confirmed full recovery by running curl -I, which returned HTTP/1.1 200 OK with the same Content-Length: 644 as before proving the exact same build was successfully restored.

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

Always back up the existing deployment before making any changes, exactly as I did here by using mv instead of rm. In real production, deploy to a versioned directory and use a symlink (e.g., /var/www/current) to switch between versions atomically  that way a failed deploy never leaves the live path empty. Add post-deployment health checks that automatically verify the site returns a 200 OK immediately after every deploy, catching failures within seconds rather than waiting for users to report them.
---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

SSH keys use asymmetric encryption  a private key stays on my machine and never travels over the network, while only the public key lives on the server. Even if someone intercepts the connection, they can't derive the private key from it. Passwords, by contrast, are transmitted during login and can be brute-forced, guessed, or leaked. A .pem key file is also much longer and more complex than any password a human would realistically choose.

---

**2. Why should only required ports be open on a production server?**

Every open port is a potential attack surface. As I saw in my own access logs, bots were probing my server within hours of it going live  scanning for login pages, .env files, and known vulnerabilities. If unnecessary ports were open, those scans could find exploitable services. Keeping only port 22 (SSH) and port 80 (Nginx) open limits what an attacker can even reach.

---

**3. Why is it important for Nginx to be enabled on boot?**

If the EC2 instance restarts  whether from planned maintenance, a crash, or an AWS event — Nginx won't start automatically unless it's enabled. That means the site would be down until someone manually SSH'd in and started it. Enabling it with systemctl enable nginx ensures the service comes back up on its own without any manual intervention.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Anyone who finds them can access your infrastructure  spin up resources at your cost, steal data, or destroy everything. AWS credentials exposed publicly have led to massive unexpected bills within hours as attackers spin up mining operations. A .pem key shared publicly gives anyone full SSH access to your server. Once credentials are exposed, the only safe response is to revoke and rotate them immediately.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

AWS charges for running resources even when they're idle. A t3.micro left running after the Free Tier expires costs money every hour. Beyond cost, abandoned running instances are a security risk — they're still publicly reachable, still being probed by bots, but no longer actively monitored. Terminating unused resources reduces both your bill and your attack surface.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

[Paste your LinkedIn post URL here:](https://lnkd.in/p/dKsSXzEA)

`Add your URL here`

---

#### Screenshot — Published LinkedIn post


[Assignment 03Screenshot Task7-2](screenshots/linkedin2.png)


---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ✅] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [✅ ] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [ ✅] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [✅ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [✅ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [✅ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [ ✅] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [✅ ] Task 8: Security & Reliability Notes answered
- [✅ ] LinkedIn post published and URL submitted
- [✅ ] Full Name visible in all required screenshots
- [✅ ] No sensitive data exposed

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