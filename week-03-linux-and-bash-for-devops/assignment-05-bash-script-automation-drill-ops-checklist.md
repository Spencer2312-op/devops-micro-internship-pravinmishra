# Assignment 5 — Bash Script Automation Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will practice Bash scripting by building a series of small automation scripts covering environment setup, variables, arrays, loops, file conditionals, if-else logic, and functions. These scripts form the foundation of real-world Linux automation used in DevOps, cloud, and production support environments.

---

# Task 1 — Bash Environment & Workspace Setup

## Goal

Verify that Bash is available on your system and create a clean workspace for this assignment.

### Evidence

#### Screenshot 1 — Output of `echo $SHELL` and `bash --version`


[Assignment 05Screenshot Task1](screenshots/05-week-03-linux-for-devops-task1-1.png)
---

#### Screenshot 2 — Output of `pwd` and `ls -lah` showing the scripts directory

[Assignment 05Screenshot Task1](screenshots/05-week-03-linux-for-devops-task1-2.png)

---

### Notes

Answer the following in your own words:

**1. What is Bash?**

Bash (Bourne Again Shell) is a command-line interpreter that lets you interact with a Linux system by typing commands. It also lets you write scripts  files containing sequences of commands that run automatically, which is what makes it powerful for automation in DevOps.

---

**2. What is the difference between shell and Bash?**

A shell is the general term for any program that takes commands from a user and passes them to the operating system. Bash is a specific type of shell  one of the most widely used on Linux systems. 
---

**3. Why is it important to confirm the Bash version before writing scripts?**

Some Bash features (like certain array syntax or string operations) only work in newer versions. Confirming the version upfront means you know exactly what features are available on your system, so your scripts won't fail due to version incompatibility especially important when writing scripts meant to run on multiple servers.

---

# Task 2 — Your First Bash Script

## Goal

Create your first Bash script, make it executable, and run it from the terminal.

### Evidence

#### Screenshot 1 — Content of `first-script.sh`

[Assignment 05Screenshot Task2](screenshots/05-week-03-linux-for-devops-task2-1.png)

---

#### Screenshot 2 — Output of `./first-script.sh`

[Assignment 05Screenshot Task2](screenshots/05-week-03-linux-for-devops-task2-2.png)

---

#### Screenshot 3 — Output of `ls -l first-script.sh` showing executable permission

[Assignment 05Screenshot Task2](screenshots/05-week-03-linux-for-devops-task2-3.png)
---

### Notes

Answer the following in your own words:

**1. What is the purpose of `#!/bin/bash`?**

It's called a shebang line  it tells the operating system which interpreter to use to run the script. Without it, the system might not know the file is a Bash script and could try to run it with the wrong interpreter or fail entirely.

---

**2. Why do we use `chmod +x` before running a script?**

By default, newly created files on Linux don't have execute permission  they can be read and written but not run as programs. chmod +x adds the execute permission, which is what allows you to run the file directly with ./script.sh.

---

**3. What is the difference between running a script using `./script.sh` and `bash script.sh`?**

./script.sh runs the script using whatever interpreter is specified in the shebang line (#!/bin/bash) and requires execute permission to be set. bash script.sh explicitly calls the Bash interpreter to run the file, bypassing the shebang line and not requiring execute permission. Both produce the same result here, but ./script.sh is the more standard way to run scripts in production.
---

# Task 3 — Variables: User Information Script

## Goal

Use variables to store and display user-related information.

### Evidence

#### Screenshot 1 — Content of `user-info.sh`

[Assignment 05Screenshot Task3](screenshots/05-week-03-linux-for-devops-task3-1.png)

---

#### Screenshot 2 — Output of `./user-info.sh`

[Assignment 05Screenshot Task3](screenshots/05-week-03-linux-for-devops-task3-2.png)
---

### Notes

Answer the following in your own words:

**1. What is a variable in Bash?**

A variable is a named container that stores a value like text, a number, or a file path  so you can reuse it throughout a script without typing the same thing repeatedly. For example, instead of typing my full name every time, I stored it in NAME and referenced it with $NAME.

---

**2. Why should we avoid spaces around the `=` sign when creating variables?**

Bash is very strict about syntax  if you write NAME = "Spencer" with spaces, Bash interprets NAME as a command and = as an argument, which throws an error. The correct syntax is NAME="Spencer" with no spaces on either side of the =.

---

**3. How do you access the value stored inside a Bash variable?**

By putting a $ sign in front of the variable name  for example, $NAME or ${NAME}. The $ tells Bash to substitute the variable's value at that point in the script rather than treating it as plain text.

---

# Task 4 — Arrays & Loops: Tools Checklist Script

## Goal

Use arrays and loops to print a checklist of tools used in Bash scripting.

### Evidence

#### Screenshot 1 — Content of `tools-checklist.sh`

[Assignment 05Screenshot Task4](screenshots/05-week-03-linux-for-devops-task4-1.png)
---

#### Screenshot 2 — Output of `./tools-checklist.sh`

[Assignment 05Screenshot Task4](screenshots/05-week-03-linux-for-devops-task4-2.png)

---

### Notes

Answer the following in your own words:

**1. What is an array in Bash?**

An array is a variable that can hold multiple values at once instead of just one. In this script, tools is an array holding seven tool names all stored under one variable name rather than creating seven separate variables.

---

**2. Why are arrays useful in scripts?**

They let you group related items together and process them all with a single loop, rather than writing repetitive code for each item. If I wanted to add a new tool to the checklist, I just add it to the array  the rest of the script handles it automatically.
---

**3. What does `"${tools[@]}"` mean?**

tools is the array name, [@] means "all elements in the array", and the ${} syntax expands it. The quotes around it ensure each element is treated as a single item even if it contains spaces. Together, "${tools[@]}" gives you every item in the array, one at a time.

---

**4. What is the purpose of the `for` loop in this script?**

The for loop goes through each item in the tools array one by one and runs the echo command for each one. Without the loop, I'd have to write a separate echo line for every single tool  the loop handles all seven (or however many) automatically in just three lines.

---

# Task 5 — Loops: Number Counter Script

## Goal

Use loops to repeat a task multiple times.

### Evidence

#### Screenshot 1 — Content of `counter.sh`

[Assignment 05Screenshot Task5](screenshots/05-week-03-linux-for-devops-task5-1.png)

---

#### Screenshot 2 — Output of `./counter.sh`


[Assignment 05Screenshot Task5](screenshots/05-week-03-linux-for-devops-task5-2.png)


---

### Notes

Answer the following in your own words:

**1. What is a loop?**
A loop is a programming construct that repeats a block of commands multiple times. Instead of writing the same command five times, you write it once inside a loop and tell it how many times to run.

---

**2. Why do we use loops in Bash scripting?**

Loops eliminate repetitive code and make scripts more efficient and maintainable. In DevOps, loops are used constantly  for example, checking the status of multiple servers, processing a list of files, or retrying a command until it succeeds.

---

**3. How many times did the loop run in your script?**

The loop ran 5 times, once for each number in the range {1..5}, printing Count: 1 through Count: 5.

---

**4. What would you change if you wanted the loop to run 10 times?**

Change {1..5} to {1..10} — that's the only change needed. The loop would then run 10 times, printing Count: 1 through Count: 10.

---

# Task 6 — Files & Conditionals: File Validation Script

## Goal

Use file checks and conditionals to verify whether files and directories exist.

### Evidence

#### Screenshot 1 — Output of `ls -lah ../test-folder`


[Assignment 05Screenshot Task6](screenshots/05-week-03-linux-for-devops-task6-1.png)


---

#### Screenshot 2 — Content of `file-check.sh`
[Assignment 05Screenshot Task6](screenshots/05-week-03-linux-for-devops-task6-2.png)
---

#### Screenshot 3 — Output of `./file-check.sh`

[Assignment 05Screenshot Task6](screenshots/05-week-03-linux-for-devops-task6-3.png)

---

### Notes

Answer the following in your own words:

**1. What does `-d` check in Bash?**

-d checks whether a given path exists and is a directory. If the directory exists it returns true, if it doesn't exist or is a file instead, it returns false.

---

**2. What does `-f` check in Bash?**

-f checks whether a given path exists and is a regular file. It returns true only if the file exists and is not a directory or other special file type.

---

**3. Why should file and directory paths be stored in variables?**

Storing paths in variables at the top of the script makes them easy to update in one place if the path ever changes, rather than hunting through the entire script to find every hardcoded reference. It also makes the script more readable — $FILE is clearer than a long path repeated multiple times.
---

**4. What happens if the file does not exist?**

The -f condition returns false, so the else branch runs instead  printing "File NOT found" followed by the path. The script doesn't crash or throw an error, it handles the missing file gracefully and continues running.

---

# Task 7 — Conditionals: Pass or Retry Script

## Goal

Use if-else conditionals to make decisions based on a variable value.

### Evidence

#### Screenshot 1 — Content of `score-check.sh` with `score=85`

[Assignment 05Screenshot Task7](screenshots/05-week-03-linux-for-devops-task7-1.png)

---

#### Screenshot 2 — Output showing `Result: Pass`

[Assignment 05Screenshot Task7](screenshots/05-week-03-linux-for-devops-task7-2.png)

---

#### Screenshot 3 — Content of `score-check.sh` with `score=55`

[Assignment 05Screenshot Task7](screenshots/05-week-03-linux-for-devops-task7-3.png)

---

#### Screenshot 4 — Output showing `Result: Retry`

[Assignment 05Screenshot Task7](screenshots/05-week-03-linux-for-devops-task7-4.png)

---

### Notes

Answer the following in your own words:

**1. What is the purpose of if-else in Bash?**

if-else lets a script make decisions it checks whether a condition is true and runs one block of code if it is, or a different block if it isn't. This is what makes scripts intelligent rather than just running the same commands every time.

---

**2. What does `-ge` mean?**
-ge stands for "greater than or equal to"  it's a numeric comparison operator in Bash. So [ $score -ge 70 ] checks whether the value of score is 70 or higher.
---

**3. Why should conditions be tested with different values?**

Testing with different values confirms the script behaves correctly in all scenarios  not just the happy path. I tested with 85 (above the threshold) and 55 (below it) to verify both the Pass and Retry branches actually work as expected.
---

**4. How can conditionals help in automation scripts?**

Conditionals allow scripts to respond differently based on real conditions  for example, only restarting a service if it's down, only sending an alert if disk usage is above 80%, or skipping a step if a file already exists. This is what makes automation scripts genuinely useful in production rather than just running blindly.

---

# Task 8 — Functions: Final Bash Automation Script

## Goal

Create a final Bash script using functions to organize reusable code.

### Evidence

#### Screenshot 1 — Content of `final-automation.sh`

[Assignment 05Screenshot Task8](screenshots/05-week-03-linux-for-devops-task8-1.png)

---

#### Screenshot 2 — Output of `./final-automation.sh`
[Assignment 05Screenshot Task8](screenshots/05-week-03-linux-for-devops-task8-2.png)

---

#### Screenshot 3 — Output of `ls -lah` showing all created scripts

[Assignment 05Screenshot Task8](screenshots/05-week-03-linux-for-devops-task8-3.png)
---

### Notes

Answer the following in your own words:

**1. What is a function in Bash?**

A function is a named block of reusable code that you define once and can call multiple times anywhere in a script. Instead of repeating the same commands in multiple places, you wrap them in a function and just call the function name whenever you need them.

---

**2. Why are functions useful in scripts?**

Functions keep scripts organized, readable, and maintainable. If something needs to change, you update it in one place rather than hunting through the entire script. They also make large scripts easier to debug since each function has a single, clear responsibility.
---

**3. Which functions did you create in this script?**

I created four functions: print_info (displays user details), print_tools (loops through and prints the tools array), check_score (uses a conditional to determine pass or retry), and check_file (checks whether a file exists on disk).

---

**4. How does this final script combine variables, arrays, loops, conditionals, files, and functions?**

It uses variables to store personal details, an array to hold the tools list, a for loop inside print_tools to iterate through the array, an if-else conditional inside check_score to evaluate the score, a file check inside check_file using -f, and functions to organize all of it into clean, reusable blocks — all called from a simple main section at the bottom.
---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

https://lnkd.in/p/dTDTUWSi

---

#### Screenshot — Published LinkedIn post

[Assignment 05Screenshot](screenshots/linkedin3.png)
---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- All script files must be created and run successfully
- Required notes must be answered clearly for every task
- Do not expose sensitive information (keys, passwords, credentials)

---

# Completion Checklist

- [ ✅] Task 1: Environment setup verified, workspace created (Screenshots 1–2, Notes answered)
- [✅ ] Task 2: First script created, executed, permissions verified (Screenshots 1–3, Notes answered)
- [✅ ] Task 3: Variables script created and run (Screenshots 1–2, Notes answered)
- [✅ ] Task 4: Arrays and loops script created and run (Screenshots 1–2, Notes answered)
- [✅ ] Task 5: Counter loop script created and run (Screenshots 1–2, Notes answered)
- [ ✅] Task 6: File validation script created and run (Screenshots 1–3, Notes answered)
- [✅ ] Task 7: Pass/Retry conditional script tested with both values (Screenshots 1–4, Notes answered)
- [ ✅] Task 8: Final automation script created and run (Screenshots 1–3, Notes answered)
- [✅ ] All scripts run without errors
- [✅ ] Full Name visible in all required screenshots
- [✅ ] LinkedIn post published and URL submitted
- [✅ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*