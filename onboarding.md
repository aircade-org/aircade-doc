> [< back](./README.md)

# AirCade OnBoarding

## Prerequisites Overview

To work on this project on Windows you need:

- A supported Windows 10/11 machine with admin permissions and internet access.
- **Git** (for cloning and version control).
- **Node.js (LTS)**, which includes npm.
- **pnpm** (package manager used by the monorepo).
- A code editor such as **Visual Studio Code** (recommended).

***

## Step 1 – Install Git

1. Go to the official Git download page and download **Git for Windows**.
2. Run the installer and follow the setup wizard, keeping default options unless you have specific needs.
3. After installation, open **Command Prompt**, **PowerShell**, or **Git Bash** and verify:  

   ```bash
   git --version
   ```

   You should see a version number (e.g., `git version 2.x.x`).

***

## Step 2 – Install Node.js (LTS)

1. Open the official Node.js download page and download the **LTS (Long Term Support)** Windows installer (`.msi`).
2. Run the installer:  
   - Accept the license agreement.  
   - Keep default installation path.  
   - Ensure **“Install npm”** is checked.
3. When the setup finishes, open a new terminal (PowerShell or Command Prompt) and verify:  

   ```bash
   node -v
   npm -v
   ```

   Both commands should print version numbers, confirming Node.js and npm are installed.

> Recommendation: Use the latest **LTS** version rather than “Current” for better stability in production projects.

***

## Step 3 – Install pnpm

`pnpm` is the package manager used for this Turborepo-based monorepo.

1. With Node.js and npm installed, run in a terminal:  

   ```bash
   npm install -g pnpm
   ```

   This installs pnpm globally on your system.
2. Once the command finishes, verify the installation:  

   ```bash
   pnpm -v
   ```

   You should see a pnpm version number printed.
3. If `pnpm` is not recognized, ensure your global npm bin directory is included in your `PATH` (usually under `C:\Users\<YourUser>\AppData\Roaming\npm` on Windows).

***

## Step 4 – Clone the Repository and Install Dependencies

1. Open a terminal (PowerShell, Command Prompt, or Git Bash).  
2. Navigate to the directory where you want to keep the project and clone the repo:  

   ```bash
   git clone <REPO_URL>
   cd <REPO_FOLDER>
   ```

3. Install all workspace dependencies using pnpm:  

   ```bash
   pnpm install
   ```

   This will read the workspace configuration (e.g., `pnpm-workspace.yaml`) and install dependencies for all apps and packages efficiently.

***

## Step 5 – Verify Your Setup

With dependencies installed, run the dev scripts defined for the monorepo:

```bash
pnpm dev
```

- This typically starts the **Next.js** frontend and **NestJS** backend concurrently via Turborepo tasks (exact commands may vary depending on the repo).  
- Open the URLs shown in the terminal (commonly `http://localhost:3000` for web and `http://localhost:3001` for API) in your browser to confirm everything runs correctly.  

If you see build or type errors, confirm that:

- `node -v`, `pnpm -v`, and `git --version` all output versions without errors.
- Your terminal was opened **after** installing Node.js and pnpm so that `PATH` changes were picked up.

> [< back](./README.md)
