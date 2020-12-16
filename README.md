# test-upstream-github
A sample project that uses Github Actions CI/CD to automatically push commits from a Github repository to GitLab. This method requires maintainer priviledges in both the Github and Gitlab repositories.

# Instructions

1. Create a Gitlab access token with write_repository access permissions. Copy the key.
2. In your GitHub project, go to settings->secrets and add the key that you copied as a secret. Name it something like: GITLAB_PUSH_KEY
3. Now, go to Gitlab and create a new repository. You can either create a new repository or clone your Github repo. In this example, I am using https://code.il2.dsop.io/chkodama/test-upstream-github
3. Next, go back to Github and create the YML file. You can either do this by going to the 'Actions' section and starting a blank action, or you can make a blank file `gitlab.yml` and put it at /.github/workflows/gitlab.yml, making the folder structrure if it's not already there. Here's what you want in the contents of the file: 

```
name: Push to GitLab CI

on: [push]

jobs:
  build:
    env:
      BRANCH: master
      DEST_BRANCH: github
      GITLAB_REPO: 'https://code.il2.dsop.io/chkodama/test-upstream-github.git'
      GITLAB_KEY: ${{ secrets.GITLAB_PUSH_KEY }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with: 
          ref: "${{ env.BRANCH }}"
          fetch-depth: 0
      - name: prep and push mirror
        run: |
          echo 'echo ${{ env.GITLAB_KEY }}' > ./creds.sh
          chmod 770 ./creds.sh
          git config --global credential.helper cache
          git config --global core.askPass ./creds.sh
          git remote add mirror "${{ env.GITLAB_REPO }}"
          git push -u mirror "${{ env.BRANCH }}":"${{ env.DEST_BRANCH }}"
```

4. You will need to edit the values corresponding to BRANCH, DEST_BRANCH, and GITLAB_REPO to the correct values for you. In this example, I'm taking the 'master' branch from github and pushing changes from it into the 'github' branch in Gitlab. (You will also need to rename secrets.GITLAB_PUSH_KEY if you named the secret key something else).

5. Commit changes and it *should* work!
