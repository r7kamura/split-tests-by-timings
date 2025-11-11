# split-tests-by-timings

GitHub Action to split test files based on JUnit XML reports, as [`circleci tests split --split-by=timings`](https://circleci.com/docs/use-the-circleci-cli-to-split-tests/#split-by-timing-data).

## Usage

### Inputs

- `reports`
    - Path to directory where JUnit XML reports are stored.
    - e.g. `tmp/junit-xml-reports`
- `glob`
    - Glob pattern to search test files.
    - e.g. `spec/**/*_spec.rb`
- `index`
    - 0-based index of test node.
    - e.g. `0`
- `total`
    - Total count of test nodes.
    - e.g. `4`
- `working-directory` (optional, default: `.`)
    - Working directory to run the action.
    - e.g. `path/to/app`
- `architecture` (optional, default: `x86_64`)
    - CPU architecture for `mtsmfm/split-test` binary.
    - e.g. `aarch64`

### Outputs

- `paths`
    - Relative paths of test files in a space-separated string.
    - e.g. `spec/models/user_spec.rb spec/models/post_spec.rb`

### Example

Here is a full example that runs tests in 4 nodes.

```yaml
# .github/workflows/example.yml
on:
  push:
    branches:
      - main
  pull_request:

jobs:
  rspec:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        ci_node_index:
          - 0
          - 1
          - 2
          - 3
    steps:
      - uses: actions/checkout@v3
      - uses: ruby/setup-ruby@v1
        with:
          bundler-cache: true
    - uses: dawidd6/action-download-artifact@v2
      with:
        branch: main
        name: junit-xml-reports
        path: tmp/junit-xml-reports-downloaded
      continue-on-error: true
    - uses: r7kamura/split-tests-by-timings@v0
      id: split-tests
      with:
        reports: tmp/junit-xml-reports-downloaded
        glob: spec/**/*_spec.rb
        index: ${{ matrix.ci_node_index }}
        total: 4
    - run : |
        bundle exec rspec \
          --format progress \
          --format RspecJunitFormatter \
          --out tmp/junit-xml-reports/junit-xml-report-${{ matrix.ci_node_index }}.xml \
          ${{ steps.split-tests.outputs.paths }}
    - if: github.ref == 'refs/heads/main'
      uses: actions/upload-artifact@v3
      with:
        if-no-files-found: error
        name: junit-xml-reports
        path: tmp/junit-xml-reports
```

## Acknowledgement

This action uses [mtsmfm/split-test](https://github.com/mtsmfm/split-test) as its internal implementation.
