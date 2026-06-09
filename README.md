# Run Your Own Hadoop Cluster — Word Count Demo
### CIND119 · Introduction to Big Data - Hadoop

Welcome! In this short, hands-on activity you'll run a **real (mini) Hadoop cluster on your own laptop** and use it to count every word in a whole book — the classic "Hello World" of big data.

You do **not** need to install Hadoop, Java, or write any code. One free tool (Docker Desktop) does all the heavy lifting. Just follow the steps in order.

**Time:** about 30 minutes the first time (most of it is a one-time download). Much faster after that.

---

## What you need

- A laptop running **Windows 10/11** or **macOS**.
- About **5 GB** of free disk space and a working internet connection (for the first run).
- Roughly **8 GB of RAM** is recommended so the cluster runs smoothly.

That's it. Everything else is in this project.

---

## What's in this project (after you download it)

```
hadoop-demo/
├─ docker-compose.yml      ← the recipe that builds and starts the cluster
├─ hadoop-conf/            ← Hadoop's settings (don't change these)
├─ presentatin/            ← the CIND119-DHA-Hadoop_Lecture.pptx file used in the lecture
└─ data/                   ← the text we'll analyze (a sample book is included)
```

You won't edit any of these to do the demo — they're already set up. Data is the book `PRIDE and PREJUDICE by Jane Austen` from `https://www.gutenberg.org/files/1342/1342-0.txt`.

---

## Step 1 — Install Docker Desktop

Docker is the tool that runs the whole Hadoop setup neatly packaged, so nothing clutters your computer.

- **Windows:** go to <https://www.docker.com/products/docker-desktop/>, download **Docker Desktop for Windows**, run the installer, and restart if it asks. Then open **Docker Desktop** from the Start menu.
- **Mac:** at the same link, download **Docker Desktop for Mac** (pick **Apple chip** for M1/M2/M3, or **Intel chip** for older Macs). Open the downloaded file and drag Docker to **Applications**, then open it.

**What you'll see:** the Docker Desktop window opens and, after a minute, shows a green whale / "**Docker Desktop is running**" at the bottom-left.

> Keep Docker Desktop open and running the whole time you do this activity.
>
> *Mac with an Apple chip (M1/M2/M3): it works fine, but you may see a harmless "platform" warning and it may run a little slower. That's normal.*

---

## Step 2 — Download this project

On this project's GitHub page, click the green **Code** button, then **Download ZIP**. Save it, then **unzip** it (Windows: right-click → Extract All; Mac: double-click).

*(If you happen to know Git, `git clone <repo-url>` works too — but the ZIP is perfectly fine.)*

**What you'll see:** a folder called something like `hadoop-demo` containing `docker-compose.yml` and the `hadoop-conf` and `data` folders shown above.

---

## Step 3 — Open a command window inside that folder

This is where you'll type a few commands.

- **Windows:** open the `hadoop-demo` folder in File Explorer. Click the **address bar** at the top, type `powershell`, and press **Enter**.
- **Mac:** open the **Terminal** app, type `cd ` (with a space), then **drag the `hadoop-demo` folder** into the Terminal window and press **Enter**.

**What you'll see:** a window with text and a blinking cursor, showing the path to your `hadoop-demo` folder. This is your "command window" for the rest of the steps.

---

## Step 4 — Start the cluster

Type this and press **Enter**:

```
docker compose up -d
```

**What you'll see (first time only):** Docker downloads Hadoop — pages of text and progress bars. **This can take several minutes.** It only happens once. When it finishes you'll see a line like `Container hadoop-demo Started`.

> If `docker compose` gives an error, try `docker-compose` (with a hyphen) instead.

---

## Step 5 — Wait until it says it's ready

Type:

```
docker compose logs -f
```

**What you'll see:** lines scrolling by. After 30–60 seconds, watch for this banner:

```
=== Hadoop is UP. NameNode http://localhost:29870  YARN http://localhost:28088 ===
```

When you see it, press **Ctrl + C** to stop watching the log.

> Pressing Ctrl + C only stops the *log view* — **Hadoop keeps running** in the background. Don't worry, you didn't turn it off.

---

## Step 6 — Open the two dashboards in your browser

Open your web browser and visit these two addresses (type them exactly):

- **<http://localhost:29870>** — the **storage dashboard**. This shows your cluster's health. Click **Utilities → Browse the file system** to see stored files.
- **<http://localhost:28088>** — the **jobs dashboard**. This shows data-processing jobs as they run and finish.

**What you'll see:** two web pages load. The first shows an "Overview" with cluster info; the second shows a (currently empty) list of applications.

> If a page doesn't load right away, wait a few seconds and refresh.

---

## Step 7 — Step "inside" the cluster

Back in your command window, type:

```
docker exec -it hadoop-demo bash
```

**What you'll see:** your prompt changes to something like `bash-4.2$`. You are now typing commands *inside* the Hadoop cluster. (For the next steps, you must be "inside" — if your prompt doesn't show `bash-4.2$`, repeat this step.)

---

## Step 8 — Check that the book is there

```
hdfs dfs -ls /demo/input
```

**What you'll see:** one line listing the text file (for example `pride.txt`) with its size. That file lives inside Hadoop's storage now.

---

## Step 9 — Run the word-count job

Copy and paste this **exactly**, then press **Enter**:

```
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount /demo/input /demo/output
```

**What you'll see:** lots of status lines. The important ones to watch:

```
map 0% reduce 0%
map 100% reduce 0%
map 100% reduce 100%
...
completed successfully
```

This takes roughly 20–40 seconds. **Tip:** refresh the jobs dashboard (<http://localhost:28088>) while it runs and you'll see your job appear there.

> You just ran a program across a cluster — and you didn't write a single line of code. The word-count program comes built into Hadoop.

---

## Step 10 — See and explore the results
 
### 10A · The most common words
 
```
hdfs dfs -cat /demo/output/part-r-00000 | sort -k2 -nr | head -15
```
 
**What you'll see:** the 15 most common words and how many times each appears, most-frequent first — `the`, `to`, `of`, `and`… Common English words naturally top the list. Something like:
```
the   4340
to    4203
of    3788
and   3376
her   1905
...
```
 
You just ran a real big-data job and got an answer. Now let's poke at it the way a **data analyst** would.
 
### 10B · Ask the data some questions
 
**How many *different* words are in the book?**
```
hdfs dfs -cat /demo/output/part-r-00000 | wc -l
```
**What you'll see:** a single number — the size of the book's vocabulary (how many distinct words Hadoop found).
 
**How often does a particular word appear?** (try any word)
```
hdfs dfs -cat /demo/output/part-r-00000 | grep -iw "love"
```
**What you'll see:** the matching word(s) and their counts, e.g. `love   90`. Swap `"love"` for `"money"`, `"happy"`, or a character's name. *(You may see a few variants like `Love` or `love,` — real text is messy, which is exactly why analysts spend time cleaning data!)*
 
### 10C · See your result living in the storage dashboard *(web UI)*
 
Open **<http://localhost:29870>** → **Utilities → Browse the file system** → click into **`/demo/output`**.
 
**What you'll see:** two items — a file called **`part-r-00000`** (your answer) and an empty **`_SUCCESS`** marker (Hadoop's "the job finished cleanly" flag). Click `part-r-00000` to see its size and storage details. Your result is simply a file, safely stored in the cluster.

> **Just browse here — "Download", "Head the file", or "Tail the file" buttons will not work.** In this local Docker setup those buttons send your browser to the cluster's *internal* address (`hadoop-demo:9864`), which your laptop can't reach, so you'll get a "can't reach this page" error. That's expected and completely harmless — this dashboard is for *browsing and inspecting* your data, and you already read the actual contents back in the terminal (Steps 10A–10B). If you want to use the Web UI and these buttons anyway, you need to add `127.0.0.1 hadoop-demo` into your `C:\Windows\System32\drivers\etc\hosts` fiile.

### 10D · Run a *second* job and watch it live *(web UI)*
 
Analysts don't only *count* — they **search**. Let's use Hadoop to find the most common **capitalized words** (often names and places). Copy-paste this:
 
```
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar grep /demo/input /demo/names "[A-Z][a-z]+"
```
 
**While it runs:** switch to the jobs dashboard **<http://localhost:28088>** and refresh. **You'll watch a brand-new job appear, run, and finish** (this one actually launches two short jobs back-to-back — one to search, one to sort the results).
 
Now read what it found:
```
hdfs dfs -cat /demo/names/part-r-00000 | head -15
```
 
**What you'll see:** the most frequent capitalized words, ranked — a mix of sentence-starters (`The`, `And`, `Mr`, `Mrs`) and, near the top, the book's **main characters**: `Elizabeth`, `Darcy`, `Bingley`, `Jane`. You just discovered the lead characters of a novel without reading a single page — that's text analytics in action.
 
> Re-running this job? Clear its output first: `hdfs dfs -rm -r /demo/names`
 
**Why this matters for analysts:** swap the book for your real data and the same two moves apply — *count* becomes "units sold per product," and *search* becomes "find every transaction matching a rule." The book is just a friendly stand-in for a billion rows.
 
---

## Step 11 (optional) — Try your own text

1. Put any plain-text `.txt` file into the **`data`** folder on your laptop.
2. In your command window, leave the cluster by typing `exit`, then run:
   ```
   docker compose down
   docker compose up -d
   ```
3. Your file is loaded automatically. Repeat Steps 7–10 to count its words.

---

## When you're finished — shut it down

Inside the cluster, type `exit` to leave it. Then, in the `hadoop-demo` folder, type:

```
docker compose down
```

**What you'll see:** a couple of "Removed" lines. The cluster is now stopped and cleaned up — your files stay on your laptop, and nothing keeps running in the background. You can start again any time with `docker compose up -d`.

---

## Troubleshooting (common hiccups)

| Problem | What to do |
|---|---|
| `docker: command not found`, or "cannot connect to the Docker daemon" | Docker Desktop isn't running. Open it and wait for the green "running" message, then try again. |
| Step 4 seems stuck or very slow | The first run downloads Hadoop — give it a few minutes. It's only slow the first time. |
| The dashboards won't open in the browser | Wait about a minute after the "Hadoop is UP" banner, then refresh. Double-check you typed `http://localhost:29870` exactly. |
| Re-running Step 9 says **output already exists** | The job won't overwrite old results. First run `hdfs dfs -rm -r /demo/output`, then run Step 9 again. |
| Commands give errors like "command not found: hdfs" | You're not "inside" the cluster. Do Step 7 again — your prompt must show `bash-4.2$`. |
| "port is already allocated" | Another program is using that port. Close it (or restart your laptop) and try Step 4 again. |

---

## A 30-second glossary (plain English)

- **Hadoop** — software that stores and processes very large amounts of data by spreading the work across many computers. Here we run a tiny one-computer version to learn how it works.
- **HDFS** — Hadoop's storage system. It chops big files into pieces ("blocks") and keeps them safe.
- **MapReduce / WordCount** — the processing job. It counts how often each word appears, doing the work in parallel.
- **Docker** — a tool that runs the whole Hadoop setup inside a self-contained "container," so you don't have to install anything complicated.

**You did it if:** the two dashboards opened, and Step 10 printed a list of top words. Nice work!
