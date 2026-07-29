# Flutter-App
modern Git workflow + CI/CD pipeline with protected branches and auto-merge rules.

Overall Architecture (What we’re building)

Developer (you + friend)
        ↓
   Feature Branches
        ↓
   Pull Request → develop branch
        ↓
   GitHub Actions (build + test)
        ↓ (if success)
   Auto merge → main branch
        ↓
   Final tests run again

If anything fails → ❌ merge blocked automatically.


-------Step 1: Create GitHub Repository------------

Login in to GItHub and Create a Repository named: Flutter-App




-----Step 2: Add Your Friend as Collaborator--------

Go to:
    Settings → Collaborators → Add people
Enter friend’s GitHub username
Give Write access
👉 Now others can:
Create branches
Push code
Raise Pull Requests

--------Step 3: Create Branch Strategy------------
We will use 3 types of branches:
Main -                 We will use 3 types of branches
Develop-               Integration branch
Feature/* -            Work Branch


===We can authenticate our terminal or VS Code to push code to a GitHub repository using three primary methods: the VS Code Built-in Account Sync, SSH Keys, or a Personal Access Token (PAT).===

==Create develop branch==
git checkout -b develop
git push origin develop



-----🔒 Step 4: Protect Important Branches--------
Go to:
    Settings → Branches → Add rule
Rule for main
Enable:
✅ Require pull request before merging
✅ Require status checks to pass
✅ Require branches to be up to date
✅ Include administrators
✅ Restrict pushes (optional)

Rule for develop
Enable:
✅ Require pull request
✅ Require status checks




-------🧪 Step 5: Setup GitHub Actions (CI Pipeline)-------------

Create workflow file:
    .github/workflows/flutter-ci.yml

Add below code:
==YAML==

name: Flutter CI

on:
  push:
    branches: [develop, main]
  pull_request:
    branches: [develop, main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.22.0'

      - name: Install dependencies
        run: flutter pub get

      - name: Analyze code
        run: flutter analyze

      - name: Run tests
        run: flutter test

      - name: Build APK
        run: flutter build apk --debug


👉 This ensures:
Code compiles
No major issues
Tests pass

-----🔄 Step 6: Enable Auto-Merge (Important)---------

GitHub supports auto-merge after checks pass.
Enable it:
    Go to:
        Setting-->General
Enable:
        Allow auto-merge


---------🔁 Step 7: Auto Merge Flow----------

Workflow:
1. We create:
    feature/chat-ui

2. Push
    git push origin feature/chat-ui

3. Create PR → develop

4. GitHub Actions runs:
    If ❌ fail → cannot merge
    If ✅ pass → enable Auto Merge


------⚙️ Step 8: Optional Auto-Merge to Main---------
We can automate:
👉 If develop passes → auto PR to main

Add another workflow:
    .github/workflows/auto-merge.yml

==YAML==

name: Auto Merge to Main

on:
  push:
    branches:
      - develop

jobs:
  promote:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Create PR to main
        uses: peter-evans/create-pull-request@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          base: main
          branch: develop
          title: "Auto Merge develop → main"


--------🔙 Step 9: Failure Handling (Rollback Logic)--------

👉 GitHub already protects you automatically:
❌ If tests fail → merge blocked
❌ If build fails → PR rejected
❌ main never breaks

For rollback:
You can revert:
    git revert <commit_id>

OR
    git reset --hard <last_good_commit>

How it Gone a work:
Now we can use below commands:  
    git checkout develop
    git pull origin develop

    git checkout -b feature/<feature-name>

After work:

    git add .
    git commit -m "feature added"
    git push origin feature/<feature-name>