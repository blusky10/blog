---
title: Github Action
tags:
---

# Github Action 설정

name: CI

on:
  push:
    branches: [ master ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: pandoc/latex
    steps:
      - uses: actions/checkout@v2
      - name: creates output
        run:  sh ./build.sh
      - name: Pushes to another repository
        uses: blusky10/blog@master
        env:
          API_TOKEN_GITHUB: ${{ secrets.API_TOKEN_GITHUB }}
        with:
          source-directory: 'output'
          destination-github-username: 'blusky10'
          destination-repository-name: 'blusky10.github.io'
          user-email: blusky0910@gmail.com
          target-branch: main


https://github.com/marketplace/actions/push-directory-to-another-repository          
https://blog.hodory.dev/2019/08/23/deploy-hexo-blog-with-github-actions/