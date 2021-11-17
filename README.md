# actions

Infracost enables you to see cloud cost estimates for Terraform in pull requests.

## Usage

Assuming you have [downloaded Infracost](https://www.infracost.io/docs/#quick-start) and run `infracost register` to get an API key, you should:

1. [Add repo secrets](https://docs.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets#creating-encrypted-secrets-for-a-repository) for `INFRACOST_API_KEY` and any other required credentials to your GitHub repo (e.g. `AWS_ACCESS_KEY_ID`).

2. By default, the latest version of the Infracost CLI is installed; you can override that using the `version` input.

    ```yml
    steps:
    - uses: infracost/actions/setup@master
      with:
        api_key: ${{ secrets.INFRACOST_API_KEY }}
        version: latest # See https://github.com/infracost/infracost/releases for other versions
    ```

3. Create a new file in `.github/workflows/infracost.yml` in your repo with the following content. Typically this action will be used in conjunction with the [setup-terraform](https://github.com/hashicorp/setup-terraform) action. The GitHub Actions [docs](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#on) describe other options for `on`, though `pull_request` is probably what you want.

```yaml
on: [pull_request]
jobs:
  infracost:
    runs-on: ubuntu-latest
    name: Post Infracost comment
    steps:
      - name: Check out repository
        uses: actions/checkout@v2

      - name: Install terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_wrapper: false

      - name: Terraform init
        run: terraform init
        working-directory: path/to/my-terraform

      - name: Terraform plan
        run: terraform plan -out plan.tfplan
        working-directory: path/to/my-terraform

      - name: Terraform show
        id: tf_show
        run: terraform show -json plan.tfplan
        working-directory: path/to/my-terraform

      - name: Save Terraform Plan JSON
        run: echo '${{ steps.tf_show.outputs.stdout }}' > plan.json # Do not change

      - name: Setup Infracost
        uses: infracost/actions/setup@master
        with:
          api_key: ${{ secrets.INFRACOST_API_KEY }}
          version: latest

      - name: Infracost breakdown
        run: infracost breakdown --path plan.json --format json --out-file infracost.json

      - name: Infracost output
        run: infracost output --path infracost.json --format github-comment --out-file infracost-comment.md

      - name: Post comment
        uses: marocchino/sticky-pull-request-comment@v2
        with:
          path: infracost-comment.md
```
