# AirCade OnBoarding

## Step 1 – Install Git

1. Go to the official Git download page and download **Git**.
2. Run the installer and follow the setup, keeping default options unless you have specific needs.
3. After installation, open **Command Prompt**, **Terminal**, or **Git Bash** and verify:  

   ```bash
   git --version
   ```

   You should see a version number (e.g., `git version 2.x.x`).

***

## Step 2 – Install Node.js (LTS)

1. Open the official Node.js download page and download the **LTS (Long Term Support)**.
2. Run the installer:  
   - Accept the license agreement.  
   - Keep default installation path.  
   - Ensure **"Install npm"** is checked.
3. When the setup finishes, open a new terminal and verify:  

   ```bash
   node -v
   npm -v
   ```

   Both commands should print version numbers, confirming Node.js and npm are installed.

> Recommendation: Use the latest **LTS** version rather than "Current" for better stability in production projects.

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
3. If `pnpm` is not recognized, ensure your global npm bin directory is included in your `PATH`.

***

## Step 4 – Install Turborepo

1. Open a terminal.
2. With pnpm installed, run in a terminal:  

   ```bash
   pnpm add turbo --global
   ```

***

## Step 5 – Clone the Repository and Install Dependencies

1. Open a terminal.  
2. Navigate to the directory where you want to keep the project and clone the repo:  

   ```bash
   git clone git@github.com:aircade-org/aircade-app.git
   cd aircade-app
   ```

3. Install all workspace dependencies using pnpm:  

   ```bash
   pnpm install
   ```

   This will read the workspace configuration (e.g., `pnpm-workspace.yaml`) and install dependencies for all apps and packages efficiently.

***

## Step 6 – Verify Your Setup

With dependencies installed, run the dev scripts defined for the monorepo:

```bash
pnpm dev
```
