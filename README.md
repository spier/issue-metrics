# Issue Metrics Action

[![CodeQL](https://github.com/github/issue-metrics/actions/workflows/codeql-analysis.yml/badge.svg)](https://github.com/github/issue-metrics/actions/workflows/codeql-analysis.yml) [![Docker Image CI](https://github.com/github/issue-metrics/actions/workflows/docker-image.yml/badge.svg)](https://github.com/github/issue-metrics/actions/workflows/docker-image.yml) [![Python package](https://github.com/github/issue-metrics/actions/workflows/python-package.yml/badge.svg)](https://github.com/github/issue-metrics/actions/workflows/python-package.yml)

This is a GitHub Action that searches for pull requests/issues/discussions in a repository and measures and reports on
several metrics. The issues/pull requests/discussions to search for can be filtered by using a search query.

The metrics that are measured are:
| Metric | Description |
|--------|-------------|
| Time to first response | The time between when an issue/pull request/discussion is created and when the first comment or review is made. |
| Time to close | The time between when an issue/pull request/discussion is created and when it is closed. |
| Time to answer | (Discussions only) The time between when a discussion is created and when it is answered. |
| Time in label | The time between when a label has a specific label appplied to an issue/pull request/discussion and when it is removed. This requires the LABELS_TO_MEASURE env variable to be set. |

This action was developed by the GitHub OSPO for our own use and developed in a way that we could open source it that it might be useful to you as well! If you want to know more about how we use it, reach out in an issue in this repository.

To find syntax for search queries, check out the [documentation on searching issues and pull requests](https://docs.github.com/en/issues/tracking-your-work-with-issues/filtering-and-searching-issues-and-pull-requests) or the [documentation on searching discussions](https://docs.github.com/en/search-github/searching-on-github/searching-discussions).

## Example use cases

- As a maintainer, I want to see metrics for issues and pull requests on the repository I maintain in order to ensure I am giving them the proper amount of attention.
- As a first responder on a repository, I want to ensure that users are getting contact from me in a reasonable amount of time.
- As an OSPO, I want to see how many open source repository requests are open/closed, and metrics for how long it takes to get through the open source process.
- As a product development team, I want to see metrics around how long pull request reviews are taking, so that we can reflect on that data during retrospectives.

## Support

If you need support using this project or have questions about it, please [open up an issue in this repository](https://github.com/github/issue-metrics/issues). Requests made directly to GitHub staff or support team will be redirected here to open an issue. GitHub SLA's and support/services contracts do not apply to this repository.

## Use as a GitHub Action

1. Create a repository to host this GitHub Action or select an existing repository.
1. Create the env values from the sample workflow below (GH_TOKEN, SEARCH_QUERY) with your information as repository secrets. More info on creating secrets can be found [here](https://docs.github.com/en/actions/security-guides/encrypted-secrets).
Note: Your GitHub token will need to have read access to the repository in the organization that you want evaluated
1. Copy the below example workflow to your repository and put it in the `.github/workflows/` directory with the file extension `.yml` (ie. `.github/workflows/issue-metrics.yml`)

### Configuration

Below are the allowed configuration options:

| field                 | required | default | description |
|-----------------------|----------|---------|-------------|
| `GH_TOKEN`            | True     |         | The GitHub Token used to scan the repository. Must have read access to all repository you are interested in scanning. |
| `SEARCH_QUERY`        | True     |         | The query by which you can filter issues/prs which must contain a `repo:` entry or an `org:` entry. For discussions, include `type:discussions` in the query. |
| `LABELS_TO_MEASURE`   | False    |         | A comma separated list of labels to measure how much time the label is applied. If not provided, no labels durations will be measured. Not compatible with discussions at this time. |
| `HIDE_TIME_TO_FIRST_RESPONSE` | False | False | If set to true, the time to first response will not be displayed in the generated markdown file. |
| `HIDE_TIME_TO_CLOSE` | False | False | If set to true, the time to close will not be displayed in the generated markdown file. |
| `HIDE_TIME_TO_ANSWER` | False | False | If set to true, the time to answer a discussion will not be displayed in the generated markdown file. |
| `HIDE_LABEL_METRICS` | False | False | If set to true, the time in label metrics will not be displayed in the generated markdown file. |

### Example workflows

#### Calculated Time Example
This workflow searches for the issues created last month, and generates an issue with metrics.

```yaml
name: Monthly issue metrics
on:
  workflow_dispatch:
  schedule:
    - cron: '3 2 1 * *'

jobs:
  build:
    name: issue metrics
    runs-on: ubuntu-latest
    
    steps:

    - name: Get dates for last month
      shell: bash
      run: |
        # Get the current date
        current_date=$(date +'%Y-%m-%d')

        # Calculate the previous month
        previous_date=$(date -d "$current_date -1 month" +'%Y-%m-%d')

        # Extract the year and month from the previous date
        previous_year=$(date -d "$previous_date" +'%Y')
        previous_month=$(date -d "$previous_date" +'%m')

        # Calculate the first day of the previous month
        first_day=$(date -d "$previous_year-$previous_month-01" +'%Y-%m-%d')

        # Calculate the last day of the previous month
        last_day=$(date -d "$first_day +1 month -1 day" +'%Y-%m-%d')

        echo "$first_day..$last_day"
        echo "last_month=$first_day..$last_day" >> "$GITHUB_ENV"

    - name: Run issue-metrics tool
      uses: github/issue-metrics@v2
      env:
        GH_TOKEN: ${{ secrets.GH_TOKEN }}
        SEARCH_QUERY: 'repo:owner/repo is:issue created:${{ env.last_month }} -reason:"not planned"'

    - name: Create issue
      uses: peter-evans/create-issue-from-file@v4
      with:
        title: Monthly issue metrics report
        content-filepath: ./issue_metrics.md
        assignees: <YOUR_GITHUB_HANDLE_HERE>

```

#### Fixed Time Example
This workflow searches for the issues created between 2023-05-01..2023-05-31, and generates an issue with metrics.

```yaml
name: Monthly issue metrics
on:
  workflow_dispatch:

jobs:
  build:
    name: issue metrics
    runs-on: ubuntu-latest
    
    steps:

    - name: Run issue-metrics tool
      uses: github/issue-metrics@v2
      env:
        GH_TOKEN: ${{ secrets.GH_TOKEN }}
        SEARCH_QUERY: 'repo:owner/repo is:issue created:2023-05-01..2023-05-31 -reason:"not planned"'

    - name: Create issue
      uses: peter-evans/create-issue-from-file@v4
      with:
        title: Monthly issue metrics report
        content-filepath: ./issue_metrics.md
        assignees: <YOUR_GITHUB_HANDLE_HERE>

```

## SEARCH_QUERY: Issues or Pull Requests? Open or closed?
This action can be configured to run metrics on discussions, pull requests and/or issues. It is also configurable by whether they were open or closed in the specified time window. Further query options are listed in the [documentation on searching issues and pull requests](https://docs.github.com/en/issues/tracking-your-work-with-issues/filtering-and-searching-issues-and-pull-requests) or the [documentation on searching discussions](https://docs.github.com/en/search-github/searching-on-github/searching-discussions). Here are some search query examples:

Issues opened in May 2023:
- `repo:owner/repo is:issue created:2023-05-01..2023-05-31`

Issues closed in May 2023 (may have been open in May or earlier):
- `repo:owner/repo is:issue closed:2023-05-01..2023-05-31`

Pull requests opened in May 2023:
- `repo:owner/repo is:pr created:2023-05-01..2023-05-31`

Pull requests closed in May 2023 (may have been open in May or earlier):
- `repo:owner/repo is:pr closed:2023-05-01..2023-05-31`

Discussions opened in May 2023:
- `repo:owner/repo type:discussions created:2023-05-01..2023-05-31`

Discussions opened in May 2023 with category of engineering and label of question:
- `repo:owner/repo type:discussions created:2023-05-01..2023-05-31 category:engineering label:"question"`

Both issues and pull requests opened in May 2023:
- `repo:owner/repo created:2023-05-01..2023-05-31`

Both issues and pull requests closed in May 2023 (may have been open in May or earlier):
- `repo:owner/repo closed:2023-05-01..2023-05-31`

OK, but what if I want both open or closed issues and pull requests? Due to limitations in issue search (no ability for OR logic), you will need to run the action twice, once for opened and once for closed. Here is an example workflow that does this: 

```yaml
name: Monthly issue metrics
on:
  workflow_dispatch:
  schedule:
    - cron: '3 2 1 * *'

jobs:
  build:
    name: issue metrics
    runs-on: ubuntu-latest

    steps:
    
    - name: Run issue-metrics tool for issues and prs opened in May 2023
      uses: github/issue-metrics:v2
      env:
        GH_TOKEN: ${{ secrets.GH_TOKEN }}
        SEARCH_QUERY: 'repo:owner/repo created:2023-05-01..2023-05-31 -reason:"not planned"'

    - name: Create issue for opened issues and prs
      uses: peter-evans/create-issue-from-file@v4
      with:
        title: Monthly issue metrics report  for opened issues and prs
        content-filepath: ./issue_metrics.md
        assignees: <YOUR_GITHUB_HANDLE_HERE>
    
    - name: Run issue-metrics tool for issues and prs closed in May 2023
      uses: github/issue-metrics:v2
      env:
        GH_TOKEN: ${{ secrets.GH_TOKEN }}
        SEARCH_QUERY: 'repo:owner/repo closed:2023-05-01..2023-05-31 -reason:"not planned"'

    - name: Create issue for closed issues and prs
      uses: peter-evans/create-issue-from-file@v4
      with:
        title: Monthly issue metrics report for closed issues and prs
        content-filepath: ./issue_metrics.md
        assignees: <YOUR_GITHUB_HANDLE_HERE>
```

## Measuring time spent in labels

**Note**: The discussions API currently doesn't support the `LabeledEvent` so this action cannot measure the time spent in a label for discussions.

Sometimes it is helpful to know how long an issue or pull request spent in a particular label. This action can be configured to measure the time spent in a label. This is different from only wanting to measure issues with a specific label. If that is what you want, see the section on [configuring your search query](https://github.com/github/issue-metrics/blob/main/README.md#search_query-issues-or-pull-requests-open-or-closed).

Here is an example workflow that does this:

```yaml
name: Monthly issue metrics
on:
  workflow_dispatch:

jobs:
  build:
    name: issue metrics
    runs-on: ubuntu-latest
    
    steps:

    - name: Run issue-metrics tool
      uses: github/issue-metrics@v2
      env:
        GH_TOKEN: ${{ secrets.GH_TOKEN }}
        LABELS_TO_MEASURE: 'waiting-for-manager-approval,waiting-for-security-review'
        SEARCH_QUERY: 'repo:owner/repo is:issue created:2023-05-01..2023-05-31 -reason:"not planned"'

    - name: Create issue
      uses: peter-evans/create-issue-from-file@v4
      with:
        title: Monthly issue metrics report
        content-filepath: ./issue_metrics.md
        assignees: <YOUR_GITHUB_HANDLE_HERE>

```

then the report will look like this:

```markdown
# Issue Metrics

| Metric | Value |
| --- | ---: |
| Average time to first response | 0:50:44.666667 |
| Average time to close | 6 days, 7:08:52 |
| Average time to answer | 1 day |
| Average time in waiting-for-manager-approval | 0:00:41 |
| Average time in waiting-for-security-review | 2 days, 4:25:03 |
| Number of items that remain open | 2 |
| Number of items closed | 1 |
| Total number of items created | 3 |

| Title | URL | Time to first response | Time to close | Time to answer | Time in waiting-for-manager-approval | Time in waiting-for-security-review |
| --- | --- | --- | --- | --- | --- | --- |
| Pull Request Title 1 | https://github.com/user/repo/pulls/1 | 0:05:26 | None | None | None | None |
| Issue Title 2 | https://github.com/user/repo/issues/2 | 2:26:07 | None | None | 0:00:41 | 2 days, 4:25:03 |

```

## Example issue_metrics.md output

Here is the output with no hidden columns:
```markdown
# Issue Metrics

| Metric | Value |
| --- | ---: |
| Average time to first response | 0:50:44.666667 |
| Average time to close | 6 days, 7:08:52 |
| Average time to answer | 1 day |
| Number of items that remain open | 2 |
| Number of items closed | 1 |
| Total number of items created | 3 |

| Title | URL | Time to first response | Time to close | Time to answer |
| --- | --- | --- | --- | --- |
| Discussion Title 1 | https://github.com/user/repo/discussions/1 | 0:00:41 | 6 days, 7:08:52 | 1 day |
| Pull Request Title 2 | https://github.com/user/repo/pulls/2 | 0:05:26 | None | None |
| Issue Title 3 | https://github.com/user/repo/issues/3 | 2:26:07 | None | None |

```

Here is the output with all hidable columns hidden:
```markdown
# Issue Metrics

| Metric | Value |
| --- | ---: |
| Number of items that remain open | 2 |
| Number of items closed | 1 |
| Total number of items created | 3 |

| Title | URL |
| --- | --- |
| Discussion Title 1 | https://github.com/user/repo/discussions/1 |
| Pull Request Title 2 | https://github.com/user/repo/pulls/2 |
| Issue Title 3 | https://github.com/user/repo/issues/3 | 2:26:07 |

```

## Example using the JSON output instead of the markdown output

There is JSON output available as well. You could use it for any number of possibilities, but here is one example that demonstrates retreiving the JSON output and then printing it out.

```yaml
name: Monthly issue metrics
on:
  workflow_dispatch:
  schedule:
    - cron: '3 2 1 * *'

jobs:
  build:
    name: issue metrics
    runs-on: ubuntu-latest

    steps:
    - name: Run issue-metrics tool
      id: issue-metrics
      uses: github/issue-metrics@v2
      env:
        GH_TOKEN: ${{ secrets.GH_TOKEN }}
        SEARCH_QUERY: 'repo:owner/repo is:issue created:2023-05-01..2023-05-31 -reason:"not planned"'

    - name: Print output of issue metrics tool
      run: echo "${{ steps.issue-metrics.outputs.metrics }}"

```

## Local usage without Docker

1. Copy `.env-example` to `.env`
1. Fill out the `.env` file with a _token_ from a user that has access to the organization to scan (listed below). Tokens should have admin:org or read:org access.
1. Fill out the `.env` file with the _search_query_ to filter issues by
1. `pip install -r requirements.txt`
1. Run `python3 ./issue_metrics.py`, which will output issue metrics data

## License

[MIT](LICENSE)
