# VxCloud Deploy Action

Deploy applications to VxCloud using GitHub Actions.

## Usage

```yaml
name: Deploy to VxCloud

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build app
        run: |
          npm ci
          npm test
          npm run build

      - name: Deploy to VxCloud
        uses: prodxcloud/vxcloud-deploy-action@v1
        with:
          api-key: ${{ secrets.VXCLOUD_API_KEY }}
          project: my-app
          environment: production
