---
layout: page
title: About
---

This is my blog.  It is nothing fancy, I used it to document some of issues I solve.
Therefore Opinions are my own and not the views of my employer.

I also have a number of upcomming posts and topics I find interesting you can see the status here: [https://trello.com/b/kxo5uCIh/pauls-blog](https://trello.com/b/kxo5uCIh/pauls-blog).

This is a jekyll site, prior to this site I had not used github actions, the ci/cd I am using is github actions it is a simple process described below.

```yaml
name: Build and deploy Jekyll site to S3
on:
  push:
    branches:
      - master

jobs:
  github-pages:
    runs-on: ubuntu-16.04
    steps:
      - uses: actions/checkout@v2
      # Use GitHub Actions' cache to shorten build times and decrease load on servers
      - uses: actions/cache@v2
        with:
          path: vendor/bundle
          key: ${ runner.os }}-gemz-${ hashFiles('**/Gemfile.lock') }
          restore-keys: |
            ${{ runner.os }}-gemz-

      - uses: lemonarc/jekyll-action@1.0.0
      
        # Push to S3
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${ secrets.AWS_ACCESS_KEY_ID }
          aws-secret-access-key: ${ secrets.AWS_SECRET_ACCESS_KEY }
          aws-region: ap-southeast-2

      - name: Sync output to S3
        run: |
          aws s3 sync ./_site/ s3://gh-paul --delete
      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation --distribution-id ${{ secrets.CF_DIST_ID }} --paths "/*"
```
The Architecture overview is so simple but still I drew it up.

I use VSCode, the markdown preview in VS code aswell as the jykell serve to build locally.

![alt text](/build.png "CockroachDB on Rasberry Pi 4")