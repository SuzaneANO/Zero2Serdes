### 1. Update your "main" branch

Before starting new work, ensure you have the latest code from the server so you don't run into merge conflicts later.

PowerShell

```
git checkout main
git pull origin main
```

### 2. Create a new "Feature Branch"

Never work directly on `main`. Create a branch specifically for the feature or fix you are working on.

PowerShell

```
# Replace 'feature-name' with something descriptive, like 'fix-ethernet-types'
git checkout -b feature-name
```

### 3. Make your changes and Stage them

After you have edited your Verilog or script files, tell Git which files you want to include in the commit.

PowerShell

```
# To add specific files
git add path/to/your/file.sv

# OR to add everything you changed
git add .
```

### 4. Commit your changes

Write a clear, concise message describing what you did.

PowerShell

```
git commit -m "Fix syntax error in GigBaseXPCS by using explicit package scoping"
```

### 5. Push the branch to GitHub

Now, send your local branch up to the remote repository.

PowerShell

```
# This tells git to link your local branch to the remote 'origin'
git push --set-upstream origin feature-name
```

### 6. The Final Step (On GitHub)

Once you run the push command, Git will usually print a URL in the terminal.

1. **Click that link** (or go to your GitHub repository page).
    
2. You will see a yellow bar saying **"feature-name had recent pushes."**
    
3. Click the green **"Compare & pull request"** button.
    
4. Add a description of your changes and click **"Create pull request."**