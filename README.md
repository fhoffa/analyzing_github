# Analyzing GitHub

A growing collection of what I know about analyzing GitHub with BigQuery and other tools

## Main datasets

### GH Archive 

A log of all public events happening on GitHub.

- https://www.githubarchive.org/
- https://bigquery.cloud.google.com/table/githubarchive:month.201808

Updated: Hourly

### GHTorrent

A relational dataset that adds more data from GitHub's knowledge graph:

- http://ghtorrent.org/gcloud.html
- https://bigquery.cloud.google.com/table/ghtorrent-bq:ght_2018_04_01.users

Updated: ~Monthly (depending on the project owner availability).

### GitHub contents

A snapshot of the open source contents of GitHub, ready to be analyzed.

- https://medium.com/google-cloud/github-on-bigquery-analyze-all-the-code-b3576fd2b150
- https://bigquery.cloud.google.com/table/bigquery-public-data:github_repos.files

Updated: ~Weekly.

-----

# FAQ

## GitHub contents

### Could you please share more information regarding which projects are available via BQ?

The `bigquery-public-data.github_repos.contents` table only contains the copy of ASCII files that are less than 10MB. To be included, projects need to be open source (as determined by GitHub's License API).

### Do you provide forked repos? or only non-forked?

Mostly non-forked.

### Which percentage of the repositories of GitHub exists on the data set?

There's many ways to count this. One of them:

    SELECT COUNT(repo_with_stars) repos_with_stars
      , 100*ROUND(COUNT(repo_in_mirror)/COUNT(repo_with_stars), 4) percentage_in_mirror
    FROM (
      SELECT DISTINCT repo_name AS repo_in_mirror
      FROM `bigquery-public-data.github_repos.files` 
    ) a RIGHT JOIN (
      SELECT repo.name AS repo_with_stars, APPROX_COUNT_DISTINCT(actor.id) stars 
      FROM `githubarchive.month.201706` 
      WHERE type='WatchEvent'
      GROUP BY 1 
      HAVING stars > 15
    ) b
    ON a.repo_in_mirror = b.repo_with_stars
    LIMIT 10

The results say that 6 months ago (June 2017) ~21,890 repositories got more than 15 stars. Of those ~21,890 repositories, 
~53.86% are mirrored on `bigquery-public-data.github_repos.contents`.

### How do you refresh the data? Do you use a selected list of projects or do you refresh the list each time?

The pipeline looks periodically for new projects. Old projects that change their license to a valid open source one (according to the API) might be missed and there's an option to add them manually.

-----

# Articles

## Misc (to be organized)

- https://medium.com/google-cloud/github-on-bigquery-analyze-all-the-code-b3576fd2b150
- https://hackernoon.com/winning-arguments-with-data-leading-with-commas-in-sql-672b3b81eac9
- https://hackernoon.com/some-coders-like-it-hot-but-most-prefer-colder-climates-4703c3f02fbb
- https://hackernoon.com/the-map-i-got-for-africa-8c8a958c686d
- https://medium.com/@hoffa/the-top-github-projects-per-country-92c275e19409
- https://medium.com/@hoffa/the-top-weekend-languages-according-to-githubs-code-6022ea2e33e8
- https://medium.com/google-cloud/sla-slo-explored-with-github-and-bigquery-e6a135919a8e
- https://medium.com/@hoffa/github-top-countries-201608-13f642493773
- https://medium.com/@hoffa/400-000-github-repositories-1-billion-files-14-terabytes-of-code-spaces-or-tabs-7cfe0b5dd7fd
- https://medium.com/google-cloud/analyzing-github-issues-and-comments-with-bigquery-c41410d3308
- https://medium.com/google-cloud/imports-in-java-from-2013-to-2016-winners-and-losers-2a9640056022
- https://medium.com/google-cloud/static-javascript-code-analysis-within-bigquery-ed0e3011732c
- https://medium.com/google-cloud/github-archive-fully-updated-notice-some-breaking-changes-64e7e7cd0967
- https://medium.com/@hoffa/the-top-contributors-to-github-2017-be98ab854e87

## Bonus: pypi top Python breakthrough install 2018/02 vs 2017/01

To be blogged... 

```
SELECT file.project, COUNT(*) pypi_201802_c, ANY_VALUE(b.rn_201701) rn_201701
FROM `the-psf.pypi.downloads201802*` a
LEFT JOIN (SELECT project, ROW_NUMBER() OVER(ORDER BY pypi_c DESC) rn_201701 FROM (
  SELECT file.project, COUNT(*) pypi_c
  FROM `the-psf.pypi.downloads201701*` 
  GROUP BY 1 ORDER BY 2 DESC
  LIMIT 10000
)) b
ON a.file.project=b.project 
WHERE b.rn_201701 IS null OR b.rn_201701>500
GROUP BY 1 ORDER BY 2 DESC
LIMIT 300
```




### Disclaimer

This is not an official Google product (experimental or otherwise), it is just
code that happens to be owned by Google.
