# GSP1078-Cloud-Run-Canary-Deployments

🚀 Fix: Cloud Run Progressive Delivery Lab (Tasks 3 & 4) – No Green Check

This guide explains how to fix the most common issues students face in:

Task 3 – Connect to GitHub, set up branch trigger, update sample application

Task 4 – Automate canary testing (master trigger + traffic split)

If you're not getting the green checkmark after pushing your changes, this guide walks you through the exact root causes and solutions.

🔎 Symptoms

You may experience one or more of the following:

Cloud Build trigger does not run after pushing new-feature-1

No new revision appears in Cloud Run

git push gcp master fails

Cloud Run does not show 90% / 10% traffic split

Qwiklabs shows no green check for Task 3 or Task 4

🧨 Root Cause #1 (Task 3) – Nested Repository Structure
What happens

Instead of this structure:

cloudrun-progression/
  app.py
  branch-cloudbuild.yaml
  master-cloudbuild.yaml


You accidentally create:

cloudrun-progression/
  cloudrun-progression/
    app.py
    branch-cloudbuild.yaml


This usually happens when:

gh repo create cloudrun-progression
mkdir cloudrun-progression
cp -r ... cloudrun-progression

Why this breaks Task 3

Cloud Build expects:

branch-cloudbuild.yaml


to be at the root of the GitHub repository.

If it's inside a subfolder, the branch trigger runs but cannot find the build config file — so no build starts.

✅ Fix for Task 3 (Clean Structure)
cd ~
rm -rf cloudrun-progression

git clone https://github.com/YOUR_USERNAME/cloudrun-progression
cd cloudrun-progression

cp -r /home/$USER/training-data-analyst/self-paced-labs/cloud-run/canary/* .


⚠️ Important: The . copies files into the current directory.

Verify:

pwd
ls


You should see:

app.py
branch-cloudbuild.yaml
Dockerfile
master-cloudbuild.yaml


directly in the root.

🧨 Root Cause #2 (Task 3 & 4) – Wrong Git Remote Name

After recloning, your remote becomes:

origin


But the lab instructions use:

git push gcp ...


This results in:

fatal: 'gcp' does not appear to be a git repository

✅ Fix – Push to Correct Remote

Check your remotes:

git remote -v


If you see:

origin  https://github.com/yourname/cloudrun-progression


Then push using:

git push origin new-feature-1
git push origin master


NOT:

git push gcp master

🧨 Root Cause #3 (Task 4) – Master Trigger Doesn’t Fire

In Task 4, you create the master trigger:

gcloud builds triggers create github --name="master" ...


Then you merge:

git checkout master
git merge new-feature-1


But Git says:

Already up to date.


That means nothing new was pushed, so the trigger never runs.

No build → no canary → no 90/10 split → no green check.

✅ Fix for Task 4 – Force a New Commit

Force a new commit on master:

echo "# trigger master build" >> README.md
git add .
git commit -m "trigger master build"
git push origin master


Now go to:

Cloud Console → Cloud Build → History

You should see a new build triggered by master.

🔍 How to Confirm Task 4 Worked

After the build completes:

Go to Cloud Run

Select hello-cloudrun

Open the Revisions tab

You should see:

90% → prod revision

10% → canary revision

0% → branch revision

If you see that traffic split, Task 4 is complete.

🧠 Why This Lab Fails for Many Students
Problem	Why It Happens	Result
Nested repo	Files copied into subfolder	Trigger cannot find cloudbuild.yaml
Wrong remote	Repo recloned	Push fails silently
No new commit on master	Merge already applied	Trigger never runs
✅ Final Checklist for Tasks 3 & 4

Before clicking Check my progress, confirm:

 branch-cloudbuild.yaml is at repo root

 master-cloudbuild.yaml is at repo root

 git remote -v shows correct remote

 A build triggered for new-feature-1

 A build triggered for master

 Cloud Run shows 90% / 10% traffic split

