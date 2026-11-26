# View the Worklog Website

The repository builds the Hugo site and publishes the generated `public` folder to the `gh-pages` branch.

## First time setup

1. Open **Settings → Pages** in the GitHub repository.
2. Set **Source** to **Deploy from a branch**.
3. Select branch `gh-pages` and folder `/ (root)`, then choose **Save**.
4. Open the **Actions** tab and run **Deploy Hugo site to GitHub Pages** with **Run workflow** if it has not run automatically.

After deployment, the site is available at:

<https://huuphuoc-nh.github.io/fcj-workshop-template/>

Vietnamese Worklog:

<https://huuphuoc-nh.github.io/fcj-workshop-template/vi/1-worklog/>

Week 1:

<https://huuphuoc-nh.github.io/fcj-workshop-template/vi/1-worklog/1.1-week1/>

## If the page still shows 404

- Confirm that the workflow completed successfully in **Actions**.
- Confirm that the `gh-pages` branch exists.
- Confirm that Pages is publishing from `gh-pages` and `/ (root)`.
- Wait a few minutes after the first deployment, then refresh the URL.
