---
layout: page
title: Dataset
---

Here we describe the step-by-step process of creating a dataset of 8,431
open-source Android apps.

The queries below are run against the [`bigquery-public-data:github_repos` dataset in Google's _BigQuery_](https://cloud.google.com/bigquery/public-data/github).
All other steps can be followed with our [open-source data collection tool](https://github.com/AndroidTimeMachine/open_source_android_apps).

![Flowchart of data collection process](/public/img/app-selection-process.png)


Finding Android Manifest Files
------------------------------

```
    SELECT
      repo_name,
      path,
      id
    FROM
      [bigquery-public-data:github_repos.files]
    WHERE
      path LIKE
        '%AndroidManifest.xml'
```

-   From table `github_repos.files`
-   Find all `AndroidManifest.xml` files
-   Store results in table `all_manifest_files`
-   Number of Rows: 378,610
-   Includes many build artifacts, included libraries, …

Extracting Package Names
------------------------

### Step 1: File content

```
    SELECT
      M.id as id,
      M.repo_name as repo_name,
      M.path as path,
      C.content as content
    FROM
      [all_manifest_files] AS M
    JOIN
      [bigquery-public-data:github_repos.contents] AS C
    ON
      M.id = C.id
    WHERE
      NOT C.binary
```

-   From table `github_repos.contents`
-   Retrieve file contents for all previously found files
-   Store results in table `all_manifest_contents`
-   Number of Rows: 378,247

### Step 2: Regular expression

```
    SELECT
      id,
      repo_name,
      path,
      REGEXP_EXTRACT(
        content,
        r'(?is)<manifest[^>]*package=[\'"]([\w\.]*)[\'"]'

      ) AS package
    FROM
      [all_manifest_contents]
    HAVING
      package IS NOT null
```

-   From table `all_manifest_contents`
-   Match `package` attribute
-   Store results in table `all_package_names`
-   Number of Rows: 378,208

### Step 3: Deduplication

```
    SELECT
      package
    FROM
      [all_package_names]
    GROUP BY
      package
```

-   From table `all_package_names`
-   Store results in table `distinct_package_names`
-   Duplicates exist because of libraries, example code, forks.
-   Number of Rows: 112,153

Filtering for Apps on Google Play
---------------------------------

-   Ping canonical Google Play link for each package name:
    `https://play.google.com/store/apps/details?id=<package_name>`
-   HTTP status code `200` is counted as verification for existence on
    Google Play.
-   Number of package names on Google Play: 9,478
-   For 2,191 additional package names status `403 Unauthorized` is returned.
    +   Some are reserved package names (e.g. for Apache
        Cordova library).
    +   Some have been pulled from Google Play.
    +   Some might not be accessible due to geo-blocking. Needs to
        be verified.
    +   Other possible reasons?
-   Some libraries have Google Play pages (e.g. Google Play Services)

Heuristic Matching of Google Play pages to GitHub repositories
--------------------------------------------------------------

With a mapping from package names to repositories from BigQuery and data from
Google Play, we try to match a unique GitHub project to each package name.

To that end a list of package names with GitHub repositories the package occurs
in is exported:

```
    SELECT
      package,
      COUNT(repo_name) AS repo_count,
      GROUP_CONCAT(repo_name) AS all_repos
    FROM
      [:manifest_files_in_github.all_package_names]
    WHERE
      package IN (
      SELECT
        package_name
      FROM
        [:manifest_files_in_github.package_names_200] )
    GROUP BY
      package
```

That list is used as input for the mapping heuristics.

This resulted in the final set of 8,431 open-source Android apps in 8,216
GitHub repositories.

## Graph database content

The results of the data collection process are a list of 8,431 open-source Android apps with metadata from their Google Play pages and 8,216 GitHub repositories with the source code of those apps.

All this information is made available in two ways:

 1. A [Neo4j graph database](/dockerImages#graph-database) containing metadata of repositories and apps and highlevel information on commit history of all repositories.
 2. [Snapshots of all GitHub repositories](/dockerImages#git-repository-snapshots) in the dataset cloned to a local Gitlab instance.

![Schema of the graph database](/public/img/dbstructure.png)

All properties of node `GooglePlayPage`
--------------------------------------

Property                   |Type             |Description
---------------------------|-----------------|---------------------------------------------------------------------------------------
docId                      |String           |Identifier of an app, `com.example.app`. This property is always present.
uri                        |String           |The URI of the Google Play page.
snapshotTimestamp          |Long             |POSIX timestamp when metadata from the Google Play entry was stored.
title                      |String           |Title of the app listing.
appCategory                |List of Strings  |A list of categories such as “Tools”.
promotionalDescription     |String           |Short description of the app.
descriptionHtml            |String           |Description of the app in original language.
translatedDescriptionHtml  |String           |Translation of `descriptionHtml` if available.
versionCode                |Int              |Numeric value of the version of the app.
versionString              |String           |Human readable version string.
uploadDate                 |Long             |POSIX timestamp of latest update of app.
formattedAmount            |String           |Price of app (“Free” or “\$1.66”)
currencyCode               |String           |Three character currency code of price (“USD”)
in-app purchases           |String           |Description of in-app purchases (“\$3.19 per item”)
installNotes               |String           |Either “Contains ads” or no value.
starRating                 |Float            |Average review between 0 and 5. May not be available if too few users have rated yet.
numDownloads               |String           |Estimated number of downloads as displayed on Google Play (e.g “10,000+ downloads”).
developerName              |String           |Name of developer.
developerEmail             |String           |Email address of developer.
developerWebsite           |String           |URI of website.
targetSdkVersion           |Int              |Android SDK version the app targets.
permissions                |List of Strings  |List of permission identifiers.

All properties of node `GitHubRepository`
----------------------------------------

Property           |Type    |Description
-------------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
id                 |Long    |Numerical identifier of this repository on GitHub.
owner              |String  |Owner name at snapshot time.
name               |String  |Repository name at snapshot time.
snapshot           |String  |URI to clone of the repository.
snapshotTimestamp  |Long    |POSIX timestamp when snapshot was taken.
description        |String  |Short description of the repository.
createdAt          |Long    |POSIX timestamp when repository has been created.
forksCount         |Int     |Number of forks from this repository created with GitHub’s *fork* functionality. Other ways of forking, cloning locally and pushing to a new repostitory are not counted.
stargazersCount    |Int     |Number of GitHub users having starred this repository.
subscribersCount   |Int     |Number of GitHub subscribers.
watchersCount      |Int     |Number of users watching this repository.
networkCount       |Int     |Number of repositories forked from same source.
ownerType          |String  |Account type of the owner, either “User” or “Organization”.
parentId           |Long    |Id of parent repository if this is a fork, otherwise -1.
sourceId           |Long    |Id of ancestor repository if this is a fork, otherwise -1.
