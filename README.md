# Allure reporting using GitHub Pages

This is an action to generate and publish Allure report with history using GitHub Pages as the web page platform.

The action does the reporting using following high-level steps:

1. Checkout current contents of the [GitHub Pages branch](#branch-to-be-used-for-github-pages-gh-pages-branch)
2. Generate and deploy the new report based on [Allure results](#path-with-allure-results-allure-results) using action [telia-actions/peaceiris-gh-pages](https://github.com/telia-actions/peaceiris-gh-pages) by [keeping X reports](#test-report-history-length-keep-reports) to the [given sub-folder](#test-report-sub-folder-subfolder) with [given test environment name](#test-environment-name-test-env-name)
3. Returns the [GitHub Pages URL](#action-output) to the newly deployed Allure report as output

---
## Action inputs

### Path with Allure results, `allure-results`
The path to a directory with Allure results, usually generated by some Allure reporting plugin like for example `allure-robotframework` Robot Framework library.
```
Required: true
Default: 'allure-results'
```

### Branch to be used for GitHub Pages, `gh-pages-branch`
The branch of the repo to which the Allure reports will be generated to. This report should be set as the source for the GitHub Pages content in GitHub settings.
```
Required: true
Default: 'gh-pages'
```

### GitHub Token, `gh-token`
GitHub Token to be used for deploying pages. Use for example `${{ secrets.GITHUB_TOKEN }}` as the value.
```
Required: true
```

### Test report history length, `keep-reports`
How many last reports will be kept on the GitHub Pages. This applies only for the given sub-folder.
```
Required: false
Default: '14'
```
### Test report sub-folder, `subfolder`
This can be used for example to store reports from test runs in different test environments, or from different test sets, into separate sub-folders.
```
Required: false
Default: ''
```
### Test environment name, `test-env-name`
Name of the test environment. Will be stored to the Allure report.
```
Required: false
Default: ''
```

---
## Action output
The action has an output, which can be used later in the workflow. The output is the GitHub Pages URL of the Allure report.

---
## Example usage in a workflow

Here is an example on how the action can be used. In the example, it is assumed that Allure results are generated to `allure-results` directory on the workspace root during the `run-tests` job.

```
jobs:
  run-tests:
    name: Generate Allure Report
    runs-on: [medium, telia-managed, Linux, X64]
    environment:
      name: ${{ inputs.testEnv || 'test' }}
    outputs:
      test-env: ${{ steps.variables.outputs.test-env }}
    steps:
      - name: Set some variables
        id: variables
        run: |
          TEST_ENV=${{ inputs.testEnv }}
          echo "test-env=${TEST_ENV^^}" >> $GITHUB_OUTPUT

      ...here are the other steps for executing the tests, which will generate also the Allure result files into allure-results folder...

      - name: Archive Allure results
        uses: actions/upload-artifact@v3
        with:
          name: allure-results
          path: 'allure-results/**'

  allure-report:
    name: Generate Allure Report
    runs-on: [small, telia-managed, Linux, X64]
    needs: run-tests
    environment:
      name: ${{ inputs.testEnv || 'test' }}
      url: ${{ steps.allure-report.outputs.report-url }}

    steps:
      - uses: actions/download-artifact@v3
        with:
          name: allure-results
          path: allure-results

      - name: Allure reporting
        uses: telia-actions/allure-gh-pages@v2
        id: allure-report
        with:
          subfolder: ${{ needs.run-robot-tests.outputs.test-env }}
          gh-token: ${{ secrets.GITHUB_TOKEN }}
          test-env-name: "${{ needs.run-robot-tests.outputs.test-env }}"

      - name: Link to test report into step summary
        run: |
          echo '${{ steps.allure-report.outputs.report-url }}' >> $GITHUB_STEP_SUMMARY
```
