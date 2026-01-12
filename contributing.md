# Contributing notes 

   * [How to Contribute](#to-contribute)
   * [How to preview a PR](contributing.md#previewing-a-pr)
   * [Building Locally](#building-locally)

## To Contribute

<!-- TODO: Update to using forks -->

1. Create a fork of the [aicr-docs-edit](https://github.com/lmilechin/aicr-docs-edit) repository
1. Clone your fork of the aicr-docs-edit repository to your computer (you only need to do this once).
2. Set up remotes to your fork and to the original [aicr-docs-edit](https://github.com/lmilechin/aicr-docs-edit) repository
2. You can view the web page locally [see below](#building-locally)
3. If you've already cloned the repo and created a branch in the past (run `git status` to see what branch you are on), be sure to checkout the main branch and pull any changes before creating a new branch:
```bash
git checkout main
git pull
```
4. Create a branch for your changes. Use your initials in the branch name, for example: `git checkout -b lm/my_updates`
    1. Push this branch to the remote repository (you only need to do this once for each branch): `git push --set-upstream origin lm/my_updates`
5. Make the changes you want to make. To preview locally you can run `mkdocs serve` at the command line in the top level of the repository. See [below](#building-locally) for more info. Commit as needed and then push the branch to the repo in github:
```bash
git add editedFile.md
git commit -m "short description of update"
git push
```
6. Create a Pull Request (PR)
    1. Go back to the [aicr-docs-edit](https://github.com/lmilechin/aicr-docs-edit) repo in your browser.
    2. If you just pushed the branch you'll see a link suggesting you create a PR, click it if it's there. If it's not there, click "Pull Requests" then "New pull request", click where it says "compare: main" and select your branch and then click "Create pull request".
    3. Fill out the form with a short description of your changes and title.
    4. A few minutes after your PR is created a preview will be available. Given the PR number N, you can see the preview at https://lmilechin.github.io/aicr-docs-previews/PR/PRN. For example, for PR1, the preview is at https://lmilechin.github.io/aicr-docs-previews/PR/PR1.
7. At least one ORCD other team member should review the PR before merging. We may ask for additional changes.
8. Once the changes have been approved the PR can be merged into the main branch. The web page will be updated automatically.

If any changes have been merged into main in the meantime, you should merge these into your branch before merging it. You can do this at anytime to get any updates on the main branch.
1. Check out the main branch: `git checkout main`
2. Pull any changes from the repository: `git pull`
    1. If there were no changes feel free to go back to your branch (`git checkout mybranch`) and skip the rest
3. Go back to your branch: `git checkout mybranch`
4. Merge changes from main into your branch: `git merge main`
    1. An editor will open (likely vi) with your commit message. Unless you want to change it you can keep the default message. Exit vi with `:wq`
    2. If there are merge conflicts you'll get a message that there are conflicts. Resolve them as needed, if you need help ask!
5. This merge adds the commits to main to the end of your branch. Push them to repository with: `git push`

## Previewing a PR

When a PR is submitted or updated a Github action step will automatically try and build the PR. 

The PR action will produces and online and a downloadable preview of the changes.

#### Online preview
An online preview is automatically produced for each PR at the location https://lmilechin.github.io/aicr-docs-previews/PR/PRN, where
N is the number of the PR. Within a PR the preview link can be accessed by selecting the following sections
   * Show all checks
   * Details
   * Summary

the preview location URL is shown next to the text `URL for preview` in the Github action summary.

## Building Locally

To set up: go into the aicr-docs-edit directory in your computer and install dependencies with `pip install -r requirements.txt`. You may need to add the bin directory for your python packages to your path, depending on how you have Python set up on your computer. You can also create a virtual environment for mkdocs:

```bash
python3 -m venv ../mkdocs_env
source ../mkdocs_env/bin/activate
pip install -r requirements.txt
```

Run `mkdocs serve` to get the pages to run locally. The command will print out where to view the web page, likely something like http://127.0.0.1:8000/. Paste this URL in your browser to view. The pages will auto-update anytime you make a change and save the file.

