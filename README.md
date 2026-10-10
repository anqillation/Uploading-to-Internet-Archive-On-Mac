# Bulk Uploading Large Files to Internet Archive on macOS

A step-by-step guide for beginners on how to bulk upload large files or folders to the [Internet Archive](https://archive.org) using the official `ia` command-line tool on macOS (Intel & Apple Silicon / M-Series).

---

## Prerequisites

Before starting, ensure you have:

1. **An Internet Archive account:** [Sign up for free](https://archive.org/account/signup) if you don't have one.
2. **Terminal:** Press `Cmd + Space`, type `Terminal`, and press `Enter`.

---

## Step 1: Install Python & the `ia` CLI Tool

Modern macOS versions require isolated installation tools like `pipx` to prevent messing with system Python packages.

1. **Install Homebrew** (macOS package manager) if you don't already have it:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

```


2. **Install `pipx`:**
```bash
brew install pipx
pipx ensurepath

```


> **Note:** Close and reopen Terminal after running `pipx ensurepath` to apply the changes.


3. **Install the Internet Archive CLI:**
```bash
pipx install internetarchive

```


4. **Verify the installation:**
```bash
ia --version

```


If it prints a version number (e.g., `internetarchive 4.x.x`), you're ready!

---

## Step 2: Authenticate Your Account

Connect the CLI tool to your Internet Archive account. You only need to do this **once**.

```bash
ia configure

```

When prompted:

* Enter your **Internet Archive email address**.
* Enter your **password**.

This generates secure S3 API keys and saves them locally to `~/.config/ia.ini`.

---

## Step 3: Item Identifier

Every item on the Internet Archive sits at a unique web page:

```{[https://archive.org/details/YOUR-UNIQUE-IDENTIFIER```

Your **identifier** is a the label/title that you'll see on the search bar. 

* Must be **all lowercase** (e.g., `my-archive-2026`).
* **No spaces** or special characters (use hyphens or underscores).
* Must be **globally unique** across all of archive.org.

### How to test if an identifier is available:

Run this in Terminal before uploading:

```bash
ia metadata my-custom-identifier-2026

```

* If it returns **`{}`** or **`null`**, the identifier is **available**.
* If it returns existing item details, it's **already taken**—pick a different name.

---

## Step 4: Uploading Your Files

Use `caffeinate -i` before the command to keep your Mac's CPU and network connection awake during long uploads.

### Option A: Uploading a Single Large File

```bash
caffeinate -i ia upload YOUR-UNIQUE-IDENTIFIER /path/to/your/file.zip \
  --metadata="title:My Upload Title" \
  --metadata="mediatype:data" \
  --metadata="collection:opensource" \
  --checksum \
  --retries=10

```

Change mediatype:data into either `data`, `texts`, `movies`, `audio`, `software` depending on what you're uploading. 


### Option B: Uploading an Entire Folder (Bulk Upload)

To upload all files inside a directory to a single Internet Archive page:

```bash
caffeinate -i ia upload YOUR-UNIQUE-IDENTIFIER /path/to/your/folder/ \
  --metadata="title:My Bulk Dataset" \
  --metadata="mediatype:data" \
  --metadata="collection:opensource" \
  --checksum \
  --retries=10

```
To get the path to your folder: Option + Command, + C

Live example:

```bash
caffeinate -i ia upload my-cool-project ~/Downloads/my-cool-project.zip
--metadata="title:My Cool Project"
--metadata="mediatype:data"
 --metadata="collection:opensource"
--checksum
--retries=10

```

---

## Breakdown of Key Command Parameters

| Flag / Parameter | What It Does |
| --- | --- |
| `caffeinate -i` | Built-in macOS tool that keeps your Mac awake until the transfer finishes. |
| `ia upload` | The core command initiating the Internet Archive file transfer. |
| `YOUR-UNIQUE-IDENTIFIER` | The URL slug for your archive page (`archive.org/details/...`). |
| `--metadata="title:..."` | Sets the main human-readable display title on the web page. |
| `--metadata="mediatype:..."` | Categorizes the content (`data`, `texts`, `movies`, `audio`, `software`). |
| `--metadata="collection:opensource"` | Directs public uploads to the Community Data collection. |
| `--checksum` | Compares local and remote MD5 file hashes to ensure data integrity. |
| `--retries=10` | Tells the client to retry up to 10 times if a connection drop occurs. |

---

## Troubleshooting

### How to Cancel an Active Upload

Press **`Ctrl` + `C**` in your Terminal window to terminate the upload immediately.

### Resuming Interrupted Uploads

If your Wi-Fi cuts out mid-transfer, re-run the **exact same upload command**. The `ia` tool automatically compares local files against remote hashes and skips files that were already successfully sent.

### Fixing `dquote>` Errors

If your Terminal prompt changes to `dquote>`, you forgot a closing quote (`"`) in one of your `--metadata` arguments. Press **`Ctrl` + `C**` to clear the prompt and fix your command syntax.
