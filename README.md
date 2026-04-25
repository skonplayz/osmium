# How to set up Osmium to begin using it

## 1. Installation & Environment Setup

### Install the Latest Version
Download the latest Osmium executable from the [Releases](https://github.com/skonplayz/osmium/releases) page.

### Adding to PATH (Efficient Method)
To run `osmium` commands from any terminal, you must add the folder containing the executable to your Environment Variables.

#### Option A: User PATH (Recommended)
* **Best for:** Personal use on a shared machine or if you do not have Admin rights.
* **Privileges:** **No Administrator privileges required.**
1. Copy the path to your Osmium folder.
2. Open the start menu and search up `env`
3. Select `Edit environment variables for your account`
4. Select `PATH`, then `Edit`
<img width="610" height="572" alt="image" src="https://github.com/user-attachments/assets/cad97069-f9a1-42ae-8142-4812e28b46ca" />
5. After clicking on `edit`, you should see a window that looks like this. Click `New` and paste the path to your Osmium folder you copied earlier.
<img width="517" height="494" alt="image" src="https://github.com/user-attachments/assets/d5c803ef-c376-40c5-b546-21ed10c8a3ed" />

#### Option B: System-wide PATH
* **Best for:** Making the tool available to every user account on the PC.
* **Privileges:** **Requires Administrator privileges.**
1. Press `Win + S`, search for "Environment Variables," and select **Edit the system environment variables**.
2. Click **Environment Variables** at the bottom right.
3. Under **System variables**, find **Path**, click **Edit**, then **New**.
4. Paste the folder path and click **OK**.

*Note: Restart any open terminals (or VS Code) for the changes to take effect.*

---

## 2. Project Initialization


1. **Create a Project Folder:** Create a new folder (e.g., `My Osmium App`).
2. **Open Terminal:** Right-click inside the folder and select **Open in Terminal**.
3. **Initialize:** Run the following command to generate a template for project files:
   ```bash
   osmium init
   ```

## 3. Coding & Customization
1. Open with VS Code: Right-click in the folder and select Open with Code (or your preferred code editor).

Edit master.osd: Define your logic within the script. For example:

```julia
use cli

dollars = 4

onstart {
    cli.print("Starting program...")
    dollars = (dollars * 5)
    cli.print("You have", dollars, "dollars.")
}
```
Save your changes (Ctrl + S).

## 4. Running & Building
To execute your code and see the output in the console:

```bash
osmium run
```
### Build the App
To compile your project for production/distribution:

```bash
osmium build
```
