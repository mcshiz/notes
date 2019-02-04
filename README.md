# notes
---

- [Use ssh keys for github](#github-ssh)
- [Mandatory code reviews](#mandatory-code-reviews)

---
<a name="github-ssh"></a>
### Use ssh keys for github
Create ssh key on your maching 
- `ssh-keygen -t rsa`

Add public key to github repo and enable
- Go to github/<YOUR REPO>/settings/SSH and GPG Keys  
- Copy public key contents from step 1 <NAME OF KEY>.pub into proper text box
- enable key

Create ssh alias
- make or edit this file ~/.ssh/config
    ```
    Host <ALIAS OF YOUR CHOICE: github-work>
    Hostname github.com
    PreferredAuthentications publickey
    IdentityFile <PATH TO PRIVATE KEY>
    User git
    ```

Change github repo urls from https to SSH
  - git remote set-url origin git@<ssh-alias><user_name>/<repo_name>.git
Use github commands
  - git pull git@<ssh-alias>:<user_name>/<repo_name>.git
      - i.e git pull git@github-work:mcshiz/happy-faces.git
<a name="mandatory-code-reviews"></a> 
### Set mandatory code reviews by specific people on all pull requests
- Create a file in the root of the project called CODEOWNERS . 
- Add github usernames to it - i.e @your_mom 
- Commit and push that file  
- Log into github and navigate to the repo, go to settings > branches > add rule
- Change branch to `staging`
- Click the "Require pull request reviews before merging" button
- Check the "Require review from Code Owners" box
