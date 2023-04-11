# First GitHub Action

Contents:

- [What are we doing today?](#what-are-we-doing-today)
- [What are GitHub Actions?](#what-are-github-actions)
- [Prerequisites](#before-we-begin)
- [Let's get started!](#lets-get-started)
- [Appendix](#appendix)

## What are we doing today?

- Understanding GitHub Actions
- Getting familiar with the syntax used to build a GitHub Action
  - We will write a workflow to download earthquake data published by USGS, which is updated every minute
- Going over a few use cases

## What are GitHub Actions?

📝 set of instructions (aka, a workflow)

🏃 that run on a virtual computer (aka, a runner)

⏱ triggered on an event — such as a schedule, a `pull` or `push` or manually by pushing a button

📥 have the ability to log files to the repository

## Prerequisites

- Let's create a new repository
- Let's change its settings so that GitHub Actions can commit and push files

## Let's get started!

### 1️⃣ Hello, data (Watch!)

To get familiar with logging data to the repository, let's write a simple `curl` script that fetches earthquake data provided by USGS. This file is updated every minute, so it's a good candidate for running a GitHub Action.
The file source page of the CSV is [here](https://earthquake.usgs.gov/earthquakes/feed/v1.0/csv.php).

Let's write a script that

1. is triggered on a schedule (every 10 minutes) or on a manual button push
2. runs on an Ubuntu virtual machine
3. uses `curl` to download a CSV
4. commits and pushes the CSV to our repo

For the purpose of this presentation, we will be working entirely in the browser. However, you can set up a local folder for version control with git.

1. Create the `.github/workflows/` directories, and a file called `hello_data.yml` inside it

   <details>

   <summary>How?</summary>

   ![](screenshots/Screenshot%202023-04-10%20at%202.17.28%20PM.png)
   </details>

2. Next, copy and paste the following code in the file, and commit it

```
# ☝️ First, a name for our Action. This will be displayed in the GitHub Actions tab.
name: Hello, data!

# ⚙️ Next, give it a trigger -- when do you want the action to run? On a push, pull, schedule?
on:
  # ⌛️ We want to schedule it
  schedule:
    # "cron" is your Unix-like operating systems' way of interpreting time. It's an expression made of five fields which represent the time to execute a command.
    #  More information here: https://crontab.guru/#*_*_*_*.
    - cron: "*/10 * * * *"
    # The keyword `workflow_dispatch` will let us manually run the Action on GitHub.
    # Without this keyword, we would have to wait for 10 minutes every time
    # to test and see how the action turned out.
  workflow_dispatch:

# 💼 Finally, let's tell our file what command to execute. A GitHub Action can be made up of several such jobs.
jobs:
  # Give this particular job a name, `hello_world`.
  get-data:
    # Tell the GitHub Action what kind of virtual machine to run on. In our case, we'll be using the latest version of Ubuntu.
    runs-on: ubuntu-latest
    # Within this job, let's define our commands in steps

    steps:
      # First, we need to clone the repo on the Ubuntu virtual runner.
      # This is just like you would clone a repo on your computer before exploring the code within it.
      - name: check out the repo
        uses: actions/checkout@v3
      # Next, we will execute a simple command, that uses curl to get the CSV
      - name: fetch data
        run: |-
          curl "https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.csv" -o usgs_current.csv
      # Lastly, after the Action has run on a virtual machine, we will need to commit and push the file back to
      # this repo, so we can access it there.
      - name: Commit and push
        run: |-
          git config user.name "Automated"
          git config user.email "actions@users.noreply.github.com"
          git add -A
          timestamp=$(date -u)
          git commit -m "Latest data: ${timestamp}" || exit 0
          git push
```

<details>

<summary>How?</summary>
![](screenshots/Screenshot%202023-04-10%20at%202.26.11%20PM.png)
</details>

3. Navigate to the `Actions` tab in the repo, and run the Action.
   We are able to manually run the Action because of the `workflow_dispatch` keyword we added to our YML file.

<details>
<summary>How?</summary>

![](screenshots/Screenshot%202023-04-10%20at%202.33.20%20PM.png)

</details>

4. Let's go back to the `Code` section of our repository, and see the new CSV created!

### 2️⃣ Hello, Python (Write!)

For our second workflow, we are going to hook up a Python script to our workflow. In a new workflow file, we will add a second task to what we did in the previous step.

Right now, the USGS data we are downloading every 10 minutes is being overwritten every time our Action runs. This Python script takes a new CSV and adds its new rows to a “main” CSV that we will create.

This time, our workflow will:

1. be triggered on a schedule (every 10 minutes) or on a manual button push
2. run on an Ubuntu virtual machine
3. use `curl` to download a CSV
4. set up a Python instance to run our `get_all_data.py` script
5. commit and push the CSV from USGS, along with our `main` csv to our repo

Similar to how we created the YML file in the browser itself, we'll add this Python file to GitHub.

1. Create a `get_all_data.py` file in the root of the directory, and paste the following code in it.

```
import pandas as pd # import pandas library for data manipulation and analysis
from pathlib import Path # import path library to work with file paths

df_current = pd.read_csv('usgs_current.csv')

path = Path("usgs_main.csv")

if path.is_file() == False:
    # if false, save initial main file
    df_current.to_csv("usgs_main.csv", index = False)

else:
    # if the file already exists, save it to a dataframe and then append to a new one
    df_main_old = pd.read_csv("usgs_main.csv")
    df_main_new = pd.concat([df_main_old,df_current])

    # deduplicate based on unique id
    df_main_new_drop_dupes = df_main_new.drop_duplicates(subset = "id", keep = "first")

    # save to dataframe and overwrite the old usgs_main file
    df_main_new_drop_dupes.to_csv("usgs_main.csv", index = False)


```

   <details>
   <summary>How?</summary>

![](screenshots/Screenshot%202023-04-07%20at%2012.10.32%20PM.png)

   </details>

2. Navigate back to the repo, and we will create another YML workflow file, just like we did in step 1. This time, we will call it `hello_python.yml`.

✍️ This section will require you to fill in some blanks ✍️
Copy the next code chunk and paste it into the file:

```
name: Hello, Python!

on:
    schedule:
        - cron: ___ # ---TO FILL:  every 🔟 minutes ---
    workflow_dispatch:

jobs:
    get-data:
        runs-on: # ---TO FILL:  let's use an Ubuntu 🖥️ (the latest one!)---
        steps:
            - name: check out the repo
              uses: actions/checkout@v3
            - name: fetch data
              run: |-
                curl "https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.csv" -o usgs_current.csv
            - name: Install requirements
              run: python -m pip install ___ ___ # ---TO FILL: what Python libraries did we use? 🐼 and 🚗 ---
            - name: Run script to create main csv
              run: python ___ # ---TO FILL: what's the name of our file? 🐍---
            - name: Commit and push if it changed
              run: |-
                git config user.name "Automated"
                git config user.email "actions@users.noreply.github.com"
                git add -A
                timestamp=$(date -u)
                git commit -m "Latest data: ${timestamp}" || exit 0
                git push

```

<details>
<summary>🆘 Help!</summary>

```
name: Hello, Python!

on:
    schedule:
        - cron: "*/10 * * * *" # ---TO FILL:  every 🔟 minutes ---
    workflow_dispatch:

jobs:
    get-data:
        runs-on: ubuntu-latest # ---TO FILL:  let's use an Ubuntu 🖥️ (the latest one!)---
        steps:
            - name: check out the repo
              uses: actions/checkout@v3
            - name: fetch data
              run: |-
                curl "https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.csv" -o usgs_current.csv
            - name: Install requirements
              run: python -m pip install pandas pathlib # ---TO FILL: what Python libraries did we use? 🐼 and 🚗 ---
            - name: Run script to create main csv
              run: python get_all_data.py # ---TO FILL: what's the name of our file? 🐍---
            - name: Commit and push if it changed
              run: |-
                git config user.name "Automated"
                git config user.email "actions@users.noreply.github.com"
                git add -A
                timestamp=$(date -u)
                git commit -m "Latest data: ${timestamp}" || exit 0
                git push

```

</details>

## Appendix

- [GitHub Actions documentation](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions), including helpful resources for all the keywords the workflow uses.

- Are there examples of GitHub Actions?

  - Run a Twitter bot or a [Slack bot](https://github.com/aadittambe/slack-bot)
  - Run [web scrapers](https://github.com/aadittambe/thanksgiving-travel)
  - Validate and run tests on changes to our [custom template](https://github.com/WPMedia/generator-custom-template/blob/dev/.github/workflows/validate.yml)
  - Build [an app](https://github.com/aadittambe/aadittambe.com/blob/main/.github/workflows/deploy.yml) every time new code is pushed
  - Other [fun](https://github.com/aadittambe/kindle-cost-scraper) things!

- Are there restrictions / limitations on GitHub Actions?

  - Each job in a workflow can run for up to [6 hours](https://docs.github.com/en/actions/learn-github-actions/usage-limits-billing-and-administration) of execution time.
  - Shortest interval you can schedule a workflow is [5 minutes](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions).

- If I have an API key or a password used in my scraper or workflow, how do I keep it hidden?
  - You can store your password as a [repository secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets), and access it similar to environment variables you would normally use.
