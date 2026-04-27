# Getting Started with Osmium

Osmium is a device-based scripting runtime for building CLI tools and desktop applications. Write `.osd` scripts, test them and then build them into Windows apps

---

## Prerequisites

Osmium ships as a self-contained `.exe` so you do not need any additional software or runtimes

---

## 1. Installation

Download the latest Osmium executable from the [Releases](https://github.com/skonplayz/osmium/releases) page and place it in a folder of your choice (e.g. `C:\Osmium`).

### Adding Osmium to PATH

To use `osmium` commands from any terminal, add the folder containing the executable to your PATH.

#### Option A: User PATH (Recommended)
- You do not need admin rights

Best for personal use or machines where you don't have Administrator privileges.

1. Copy the path to your Osmium folder (e.g. `C:\Osmium`).
2. Open the Start menu and search for `env`.
3. Select **Edit environment variables for your account**.
4. Select **Path**, then click **Edit**.

   <img width="610" height="572" alt="Environment Variables window" src="https://github.com/user-attachments/assets/cad97069-f9a1-42ae-8142-4812e28b46ca" />

5. Click **New**, paste your Osmium folder path, then click **OK**.

   <img width="517" height="494" alt="Edit Path window" src="https://github.com/user-attachments/assets/d5c803ef-c376-40c5-b546-21ed10c8a3ed" />

#### Option B: System-wide PATH *(Requires Administrator privileges)*

Best for making Osmium available to all user accounts on the machine.

1. Press `Win + S`, search for **Environment Variables**, and select **Edit the system environment variables**.
2. Click **Environment Variables** at the bottom right.
3. Under **System variables**, select **Path** and click **Edit**.
4. Click **New**, paste your Osmium folder path, then click **OK**.

### Restart any open terminals (or VS Code) for PATH changes to take effect.

### Optional: Register the Windows Context Menu

Run the following command once to add an **Execute Osmium Device** right-click option for `.osd` files:

```bash
osmium register
```

---

## 2. Creating a Project

1. Create a new folder for your project (e.g. `my-osmium-app`).
2. Right-click inside the folder and select **Open in Terminal**.
3. Run the init command to generate your project files:

```bash
osmium init
```

This gives you options for what files to create. The default is a master.osd entry file and a settings.toml project config file.

---

## 3. Writing Your First Script

Open your project in VS Code (or any editor) and edit `master.osd`:

```
use cli

dollars = 4

onStart {
    cli.print("Starting program...")
    dollars = (dollars * 5)
    cli.print("You have {dollars} dollars.")
}
```

A few things to note:
- `onStart { }` is the only entry point, so nothing will run until you call it here
- `use cli` loads the CLI extension
- Variables are dynamically typed and created on assignment

---

## 4. Running & Building

**Run your project:**
```bash
osmium run
```

**Watch for changes and auto-reload:**
```bash
osmium watch
```

**Build a standalone exe for distribution:**
```bash
osmium build
```

The compiled `.exe` will be written to the `dist/` folder inside your project.

---

## ⚖️ License & Terms

Osmium is a proprietary tool developed by [skonplayz](https://github.com/skonplayz).

- **Usage:** You are free to use Osmium to create, build, and distribute your own cli apps
- **Restrictions:** You may not redistribute the Osmium binary, reverse engineer the source code, or claim the framework as your own
- **Ownership:** All rights to the Osmium engine and its associated brand are reserved
