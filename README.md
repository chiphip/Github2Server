# Github2Server
Connect Github to your Server

There are two places where we need to make changes:
1. Server
2. Github

### 1. Server

Paste the following command in your linux server:

`ssh-keygen -t ed25519 -C "YOUREMAIL@GMAIL.COM" -f ~/.ssh/id_ed25519 -N '' && cat ~/.ssh/id_ed25519 && cat ~/.ssh/id_ed25519.pub && cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys && ssh-keyscan github.com >> /root/.ssh/known_hosts && eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519`

Change "YOUREMAIL@GMAIL.COM" with **your email**

This command will do the following things in one go:

- Generate an ed25519 SSH key with your email as the comment and save it in ~/.ssh/id_ed25519.
- Display your private key.
- Display your public key.
- Append your public key to the ~/.ssh/authorized_keys file.
- Add GitHub’s host key to the known_hosts file.
- Start the SSH agent.
- Add your private key to the SSH agent.

For the Github setup we need some information from this input:
1. Your Public Key (Your command output shoud display this - starts with 'ssh-ed25519 AAAA....')
2. Your Private Key (Your command output shoud display this - starts with '-----BEGIN OPENSSH PRIAVTE KEY-----')
3. Your Server's IP address (Get it from your Host)
4. Your Server's Username (use the following command if you don't know: echo $USER)


---

### 2) Github

----Step 1 on Github----

Go to your **Repository** and then click **Settings** and click **Deploy keys**.
On the **Deploy keys** page, click **Add deploy key** button.
On the **Deploy keys / Add new** page, put the folloiwn information in:
  1. Enter a **title** (anything you like)
  2. Enter your **Public Key** that we got from the command that we did earlier on the Server.
  3. Check **Allow write access**
  4. Press **Add key**

![alt text](https://github.com/chiphip/Github2Server/blob/main/71.png?raw=true)
![alt text](https://github.com/chiphip/Github2Server/blob/main/72.png?raw=true)


----Step 2 on Github----

In your **Repository** click **Settings>Secrets and Variables>Actions**.
On the **Actions secrets and variables** page click **new repository secret** button. Here we need to repeat this step 3x times because we need to provide the following Secretes:
1. **SSH_HOST** (Looks like: 128.167.40.104)
2. **SSH_KEY** (Looks like: -----BEGIN OPENSSH PRIAVTE KEY-----

b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAA)

3. **SSH_USERNAME** (Looks like: root)



----Step 3 on Github----

In your **Repository** click **Actions**.
On the **Get started with GitHub Actions** page click **Skip this and set up a workflow yourself** link.
This will get you to the following page **YourRepository/.github/workflows/main.yml**. Put the following insturctions in here:

Remember to change this **"git@github.com:YOURUSERNAME/YOURREPOSITORY.git"** to your username and repository!

```
name: Build & Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy To Server
        uses: appleboy/ssh-action@v0.1.2
        with:
          host: ${{secrets.SSH_HOST}} # IP address of the server you wish to ssh into
          key: ${{secrets.SSH_KEY}} # Private Key of the server
          username: ${{ secrets.SSH_USERNAME }} # User of the server you want to ssh into
     
          script: |
            mkdir test 
            cd test 
            git clone git@github.com:YOURUSERNAME/YOURREPOSITORY.git
            echo 'Deployment successful to server'
```

Press on **Commit changes** and the Github Actions will for the first time connect to your Server. And anytime you push/hhange something on your repository, Github will update the changes on the Server.

