# Create your GitHub repository - step by step

This guide assumes you have a GitHub account and the ZIP package supplied with this assignment.

## 1. Extract the ZIP

1. Download the ZIP file.
2. Right-click it and choose **Extract All**.
3. Open the extracted `HAL-Dataverse-Assignment-Package` folder.
4. Open `HAL-Dataverse-Architecture`.
5. Check that you can see `README.md`, `docs` and `diagrams`.

**Important:** Do not upload the `PRIVATE-Interview-Preparation` folder to the client repository. It is for your interview preparation only.

## 2. Create a GitHub repository

1. Sign in at https://github.com.
2. Click the **+** icon in the top-right corner.
3. Select **New repository**.
4. Repository name: `HAL-Dataverse-Architecture`.
5. Description: `Dynamics 365 and Dataverse architecture proposal for HAL`.
6. Choose **Public** only if you are comfortable making the assignment public. Otherwise choose **Private**.
7. Do not add a README, .gitignore or licence because the project already contains its own README.
8. Click **Create repository**.

## 3. Upload the project through the GitHub website

This is the simplest approach if you do not use Git locally.

1. Open your new, empty repository.
2. Select **Add file** -> **Upload files**.
3. Open the extracted `HAL-Dataverse-Architecture` folder on your computer.
4. Select the contents inside that folder: `README.md`, `REPOSITORY-SETUP-GUIDE.md`, `docs` and `diagrams`.
5. Drag them into the GitHub upload area. If your browser does not preserve folders when dragging, use GitHub Desktop or the Git command line instead.
6. Wait for the upload to finish.
7. In the commit message, enter `Add HAL Dataverse architecture proposal`.
8. Click **Commit changes**.

GitHub's web upload has file and browser limitations. If the upload is awkward, use GitHub Desktop (steps below).

## 4. Alternative: use GitHub Desktop

1. Install GitHub Desktop from https://desktop.github.com/.
2. Sign in with your GitHub account.
3. Choose **File** -> **Add Local Repository** if you have already cloned the repository, or **File** -> **Clone Repository** to clone the empty GitHub repository.
4. Copy the contents of the extracted `HAL-Dataverse-Architecture` folder into the cloned repository folder.
5. In GitHub Desktop, review the changed files.
6. Enter a summary such as `Add HAL Dataverse architecture proposal`.
7. Click **Commit to main**.
8. Click **Push origin**.

## 5. Check the repository before sharing

Open the repository in your browser and verify:

- The README appears on the repository home page.
- The `docs` folder contains all six design documents.
- The `diagrams` folder contains the three Mermaid diagrams.
- The links in the README open the correct files.
- No private interview notes, personal information, credentials, tokens or client secrets have been uploaded.
- The repository visibility matches what you intend.

## 6. Grant access if the repository is private

1. Open the repository.
2. Go to **Settings** -> **Collaborators** (or **Collaborators and teams**, depending on the interface).
3. Choose **Add people**.
4. Enter the GitHub usernames or email addresses supplied by Sidelah / LogicEra.
5. Send the invitation.
6. Confirm that the intended reviewers have accepted or can access the repository.

Do not guess GitHub usernames or invite unrelated accounts. Ask the recruiter for the exact GitHub accounts if they have not supplied them.

## 7. Send the repository link

Copy the repository URL from your browser and reply to the original Sidelah email thread. Keep both Sidelah HR and LogicEra included, as requested.

Before sending, check that the repository is accessible to the reviewers. A private repository link alone does not grant access.

## 8. Useful Git commands (optional)

If Git is installed and you prefer the command line, open a terminal inside the project folder:

```bash
git init
git add .
git commit -m "Add HAL Dataverse architecture proposal"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/HAL-Dataverse-Architecture.git
git push -u origin main
```

Replace `YOUR-USERNAME` with your actual GitHub username. If GitHub asks you to authenticate, complete the browser sign-in flow. Do not put a password or access token directly into the remote URL.

## 9. Final submission checklist

- [ ] Repository created.
- [ ] Repository contains only client-facing assignment material.
- [ ] README and diagrams reviewed in GitHub.
- [ ] Private study guide excluded.
- [ ] Repository access tested.
- [ ] Both Sidelah HR and LogicEra remain included in the email reply.
- [ ] Repository link and any requested source files included.
