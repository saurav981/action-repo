# Complete Setup Guide - GitHub Webhook Project

This guide will walk you through setting up both repositories and getting the entire system working.

## Overview

You need to create TWO repositories:

1. **webhook-repo**: Flask application that receives and displays webhook events
2. **action-repo**: GitHub repository that triggers webhooks on Push/PR/Merge

## Prerequisites Checklist

Before starting, make sure you have:

- [ ] Python 3.8 or higher installed
- [ ] Git installed
- [ ] GitHub account
- [ ] MongoDB Atlas account (free tier) - [Sign up here](https://www.mongodb.com/cloud/atlas)
- [ ] ngrok account (free tier) - [Sign up here](https://ngrok.com/)

## Part 1: Set Up MongoDB Atlas

### Step 1: Create MongoDB Cluster

1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) and sign up/login
2. Click "Build a Database"
3. Choose **FREE** tier (M0 Sandbox)
4. Select your preferred cloud provider and region
5. Click "Create Cluster" (takes 3-5 minutes)

### Step 2: Create Database User

1. Click on "Database Access" in left sidebar
2. Click "Add New Database User"
3. Choose "Password" authentication
4. Set username and password (remember these!)
5. Set privileges to "Read and write to any database"
6. Click "Add User"

### Step 3: Whitelist Your IP

1. Click on "Network Access" in left sidebar
2. Click "Add IP Address"
3. Click "Allow Access from Anywhere" (for development)
4. Click "Confirm"

### Step 4: Get Connection String

1. Click on "Database" in left sidebar
2. Click "Connect" on your cluster
3. Choose "Connect your application"
4. Copy the connection string
5. It looks like: `mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/`
6. Replace `<username>` with your database username
7. Replace `<password>` with your database password
8. Add database name at the end: `mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/github_webhooks`

**Save this connection string - you'll need it later!**

## Part 2: Set Up webhook-repo (Flask Application)

### Step 1: Create GitHub Repository

1. Go to GitHub and click "New repository"
2. Name it: `webhook-repo`
3. Keep it public
4. Don't initialize with README (we already have files)
5. Click "Create repository"
6. **Save the repository URL**

### Step 2: Clone and Set Up Locally

Open terminal and run:

```bash
# Navigate to where you want to store the project
cd ~/Desktop  # or any folder you prefer

# Copy the webhook-repo folder to this location
# (You already have all the files from me)

# Navigate into the folder
cd webhook-repo

# Initialize git
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit"

# Add remote (replace with your actual repo URL)
git remote add origin https://github.com/your-username/webhook-repo.git

# Push to GitHub
git branch -M main
git push -u origin main
```

### Step 3: Set Up Virtual Environment

```bash
# Still in webhook-repo folder
pip install virtualenv
virtualenv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On macOS/Linux:
source venv/bin/activate
```

### Step 4: Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 5: Configure Environment Variables

```bash
# Copy the example env file
cp .env.example .env

# Open .env in a text editor and add your MongoDB connection string
# On Windows:
notepad .env
# On macOS:
open -e .env
# On Linux:
nano .env
```

Edit the `.env` file:

```env
MONGODB_URI=mongodb+srv://your-username:your-password@cluster0.xxxxx.mongodb.net/github_webhooks?retryWrites=true&w=majority
DATABASE_NAME=github_webhooks
COLLECTION_NAME=events
FLASK_ENV=development
SECRET_KEY=super-secret-key-change-this
```

**Important**: Replace the MONGODB_URI with your actual connection string from Part 1!

### Step 6: Test the Flask Application

```bash
# Make sure you're in webhook-repo folder with venv activated
python run.py
```

You should see:

```
 * Running on http://127.0.0.1:5000
```

Open your browser and go to `http://localhost:5000`

You should see the GitHub Webhook Monitor interface (it will show "No events yet").

**Leave this terminal running!**

## Part 3: Set Up ngrok

### Step 1: Install ngrok

1. Go to [ngrok.com](https://ngrok.com/) and sign up
2. Download ngrok for your operating system
3. Unzip/extract the downloaded file

### Step 2: Configure ngrok

```bash
# Open a NEW terminal (keep Flask running in the first one)

# Navigate to where you extracted ngrok
cd ~/Downloads  # or wherever you put it

# Add your authtoken (get this from ngrok dashboard)
./ngrok config add-authtoken YOUR_AUTH_TOKEN_FROM_NGROK_DASHBOARD
```

### Step 3: Start ngrok Tunnel

```bash
# Still in the ngrok folder
./ngrok http 5000
```

You should see:

```
Forwarding    https://abc123def.ngrok.io -> http://localhost:5000
```

**Copy the HTTPS URL** (e.g., `https://abc123def.ngrok.io`)

**Leave this terminal running too!**

## Part 4: Set Up action-repo (Trigger Repository)

### Step 1: Create GitHub Repository

1. Go to GitHub and click "New repository"
2. Name it: `action-repo`
3. Keep it public
4. Don't initialize with README
5. Click "Create repository"

### Step 2: Set Up Locally

Open a NEW terminal:

```bash
# Navigate to your projects folder
cd ~/Desktop  # same folder as before

# Copy the action-repo folder here

# Navigate into it
cd action-repo

# Initialize git
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit"

# Add remote (replace with your actual repo URL)
git remote add origin https://github.com/your-username/action-repo.git

# Push to GitHub
git branch -M main
git push -u origin main
```

## Part 5: Configure GitHub Webhook

### Step 1: Add Webhook to action-repo

1. Go to your `action-repo` on GitHub
2. Click **Settings** (top menu)
3. Click **Webhooks** (left sidebar)
4. Click **Add webhook**

### Step 2: Configure Webhook Settings

Fill in these fields:

**Payload URL:**

```
https://your-ngrok-url.ngrok.io/webhook/receiver
```

Replace `your-ngrok-url` with the URL from ngrok (Part 3, Step 3)

**Content type:**

```
application/json
```

**Secret:** Leave blank

**Which events would you like to trigger this webhook?**

- Select: "Let me select individual events"
- Check ONLY these two:
  - ✅ **Pushes**
  - ✅ **Pull requests**
- Uncheck everything else

**Active:**

- ✅ Make sure this is checked

Click **Add webhook**

### Step 3: Verify Webhook

After adding, you should see:

- A green checkmark ✓ next to your webhook
- This means the "ping" event was successful

## Part 6: Test the System

Now let's test if everything works!

### Test 1: Push Event

In your terminal (in the action-repo folder):

```bash
# Make a change
echo "Test push" >> sample.txt

# Commit and push
git add .
git commit -m "Test push event"
git push origin main
```

**Check the UI:**

1. Go to `http://localhost:5000` in your browser
2. Within 15 seconds, you should see: "YourUsername pushed to main on [date/time]"

### Test 2: Pull Request Event

```bash
# Create a new branch
git checkout -b test-feature

# Make a change
echo "Feature test" >> sample.txt

# Commit and push
git add .
git commit -m "Add feature"
git push origin test-feature
```

Then on GitHub:

1. Go to your action-repo
2. You'll see a banner "Compare & pull request" - click it
3. Click "Create pull request"

**Check the UI:**

- You should see: "YourUsername submitted a pull request from test-feature to main on [date/time]"

### Test 3: Merge Event

On GitHub:

1. In your open pull request, click "Merge pull request"
2. Click "Confirm merge"

**Check the UI:**

- You should see: "YourUsername merged branch test-feature to main on [date/time]"

## Part 7: Verify MongoDB

You can verify data is being stored:

1. Go to MongoDB Atlas
2. Click "Collections"
3. You should see:
   - Database: `github_webhooks`
   - Collection: `events`
   - Documents with your events

## Troubleshooting

### "No events appearing in UI"

**Check Flask is running:**

```bash
# Should see "Running on http://127.0.0.1:5000"
```

**Check ngrok is running:**

```bash
# Should see "Forwarding https://... -> http://localhost:5000"
```

**Check browser console:**

- Open browser Dev Tools (F12)
- Look for errors in Console tab

### "MongoDB connection error"

**Verify connection string:**

- Check `.env` file has correct MONGODB_URI
- Make sure you replaced <username> and <password>
- Make sure you added database name at the end

**Check MongoDB Atlas:**

- IP address is whitelisted
- Database user exists and has permissions

### "Webhook not working"

**Check GitHub webhook delivery:**

1. Go to action-repo → Settings → Webhooks
2. Click on your webhook
3. Check "Recent Deliveries"
4. If you see red X, read the error message

**Common issues:**

- ngrok URL in webhook doesn't match ngrok URL in terminal
- Flask app not running
- Typo in webhook URL

### "ngrok URL changed"

This happens when you restart ngrok (free tier):

1. Copy new ngrok URL
2. Go to GitHub → action-repo → Settings → Webhooks
3. Click on your webhook → Edit
4. Update Payload URL
5. Save

## Submission Checklist

Before submitting, verify:

- [ ] webhook-repo is pushed to GitHub
- [ ] action-repo is pushed to GitHub
- [ ] Both repositories are public
- [ ] MongoDB is set up and events are being stored
- [ ] UI is working and showing events
- [ ] Tested push, pull request, and merge events
- [ ] README files are in both repositories
- [ ] Both repository URLs are ready to submit

## Repository URLs to Submit

```
webhook-repo: https://github.com/your-username/webhook-repo
action-repo: https://github.com/your-username/action-repo
```

## What to Keep Running

During demonstration/testing:

1. ✅ Flask application (`python run.py`)
2. ✅ ngrok tunnel (`ngrok http 5000`)
3. ✅ MongoDB Atlas (cloud - always running)

## Production Notes

For production deployment (after assessment):

- Deploy Flask app to Heroku/Render/Railway
- Use the deployment URL instead of ngrok
- Enable webhook signature verification
- Add proper error handling and logging

## Need Help?

If stuck:

1. Check the error messages carefully
2. Read the README files in both repositories
3. Check GitHub webhook delivery logs
4. Verify all services are running
5. Double-check environment variables

## Summary

You now have:
✅ webhook-repo receiving and displaying events
✅ action-repo triggering webhooks
✅ MongoDB storing event data
✅ Real-time UI updating every 15 seconds
✅ Support for Push, Pull Request, and Merge events

Good luck with your assessment! 🚀
