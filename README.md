# First GitHub Action

## What are GitHub Actions?

📝 set of instructions (aka, a workflow)

🏃 that run on a virtual computer (aka, a runner)

⏱ triggered on an event — such as a schedule, a `pull` or `push` or manually by pushing a button

📥 have the ability to log files to the repository

## What are we doing today?

🧑‍💻 getting familiar with the syntax to build a workflow (a markup language called YML)

⚙️ run three tasks

## Before we begin:

🚓 Let's give your GitHub repo permission to accept data from a GitHub Action

---

## Let's get started

### 1️⃣ Hello, world!

To get familiar with the YML syntax, and how a basic GitHub Action works, let's write a simple script that

1. is triggered on a schedule (every 10 minutes) or on a manual button push
2. runs on an Ubuntu virtual machine
3. prints a string, `Hello, world!` using the `echo` command.

Workflows live in hidden directories called `.github/workflows`. Note the `.` before "github". Let's create that directly from the browser.

1. Create the `.github/workflows/` directories, and a file called `hello_world.yml` inside it

   <details>

   <summary>How?</summary>

   ![](screenshots/Screenshot%202023-04-07%20at%2011.34.00%20AM.png)

   </details>

2. Next, copy and paste the following code in the file, and commit it

   ```
   name: 1. Hello, world!

   on:
   schedule:
       - cron: "*/10 * * * *"
   workflow_dispatch:

   jobs:
   hello_world:
       runs-on: ubuntu-latest
       steps:
       - name: check out the repo
           uses: actions/checkout@v3
       - name: print, "hello, world!"
           run: echo hello, world!
   ```

   <details>
   <summary>How?</summary>

   ![](screenshots/Screenshot%202023-04-07%20at%2011.36.54%20AM.png)

   </details>

3. Navigate to the `Actions` tab in the repo, and see the `Hello, world` string logged!
   <details>
   <summary>How?</summary>

   ![](screenshots/Screenshot%202023-04-07%20at%2011.47.18%20AM.png)
   ![](screenshots/Screenshot%202023-04-07%20at%2011.49.06%20AM.png)

   </details>

You built your first GitHub Action

### 2️⃣ Hello, data!

To get familiar with logging data to the repository, let's write a simple `curl` script that fetches earthquake data provided by USGS. This file is updated every minute, so it's a good candidate for running a GitHub Action.

Let's write a script that

1. is triggered on a schedule (every 10 minutes) or on a manual button push
2. runs on an Ubuntu virtual machine
3. uses `curl` to download a CSV
4. commits and pushes the CSV to our repo

We'll use the instructions from 1️⃣ to create the YML workflow.

1. Let's create a new YML file, and call it `hello_data.yml`

2. Copy and paste the following YML code in that file, and commit it:

   ```
   name: 2. Hello, data!

   on:
   schedule:
       - cron: "*/10 * * * *"
   workflow_dispatch:

   jobs:
   get-data:
       runs-on: ubuntu-latest
       steps:
       - name: check out the repo
           uses: actions/checkout@v3
       - name: fetch data
           run: |-
           curl "https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.csv" -o usgs_current.csv
       - name: Commit and push
           run: |-
           git config user.name "Automated"
           git config user.email "actions@users.noreply.github.com"
           git add -A
           timestamp=$(date -u)
           git commit -m "Latest data: ${timestamp}" || exit 0
           git push
   ```

3. In the Actions tab, see the workflow run

### 3️⃣ Hello, Python!

For our last workflow, we are going to up the complexity a bit, and hook a Python script to our workflow. In a new workflow file, we will add a second task to what we did in the previous step.

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

2. Navigate back to the repo, and we will create another YML workflow file, just like we did in step 1 and 2. This time, we will call it `hello_python.yml`.

   ✍️ This section will require you to fill in some blanks ✍️

   Copy the next code chunk and paste it into the file:

   ```
   name: 3. Hello, Python!

   on:
   schedule:

       - cron: # ---TO FILL:  Every 🔟 minutes---
   workflow_dispatch:

   jobs:
   get-data:
       runs-on:: # ---TO FILL:  let's use an Ubuntu 🖥️ (the latest one!)---
       steps:
       - name: check out the repo
           uses: actions/checkout@v3
       - name: fetch data
           run: |-
           curl "https://earthquake.usgs.gov/earthquakes/feed/v1.0/summary/all_day.csv" -o usgs_current.csv
       - name: Install requirements
           run: python -m pip install # ---TO FILL: what Python library did we use? 🐼---
       - name: Run script to create main csv
           run: python # ---TO FILL: what's the name of our file? 🐍---
       - name: Commit and push if it changed
           run: |-
           git config user.name "Automated"
           git config user.email "actions@users.noreply.github.com"
           git add -A
           timestamp=$(date -u)
           git commit -m "Latest data: ${timestamp}" || exit 0
           git push
   ```

3. Go to the Actions tab and run the workflow!
