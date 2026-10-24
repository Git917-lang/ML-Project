#Conflict Notes
## What happened
I edited line 3 of README.md on both branches:
- main: "Project title: Student Grade Prediction (Main Version)"
- proposal: "Project title: Predicting Student Performance (Proposal Version)"
Git could not merge automatically and reported a merge conflict.
## How I detected it
VS Code showed README.md under "Merge Changes" with a C marker andconflict markers (<<<<<<<, =======, >>>>>>>) inside the file.
## How I resolved it
1. Opened README.md in the VS Code editor.
2. Compared the Current (proposal) and Incoming (main) versions.
3. Wrote a final title combining both: "Project title: Predicting Student Performance".
4. Removed all conflict markers.5. Staged README.md, committed the merge and pushed.
## What I learned
- Conflicts happen when the same line changes in two branches.- Always remove all markers before committing.