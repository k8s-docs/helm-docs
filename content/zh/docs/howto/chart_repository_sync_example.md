---
title: "同步您的图表库"
description: "介绍如何同步本地和远程图表的存储库。"
weight: 2
aliases: ["/docs/chart_repository_sync_example/"]
---

_注意: 这个例子是专门为服务于图表存储库中的谷歌云存储（GCS）桶。_

## 先决条件

- Install the [gsutil](https://cloud.google.com/storage/docs/gsutil) tool. _We
  rely heavily on the gsutil rsync functionality_
- Be sure to have access to the Helm binary
- _Optional: We recommend you set [object
  versioning](https://cloud.google.com/storage/docs/gsutil/addlhelp/ObjectVersioningandConcurrencyControl#top_of_page)
  on your GCS bucket in case you accidentally delete something._

## 设置本地图表库目录

Create a local directory like we did in [the chart repository
guide]({{< ref "/docs/topics/chart_repository.md" >}}), and place your packaged charts in that directory.

For example:

```console
$ mkdir fantastic-charts
$ mv alpine-0.1.0.tgz fantastic-charts/
```

## 产生一个更新的 index.yaml 中

Use Helm to generate an updated index.yaml file by passing in the directory path
and the url of the remote repository to the `helm repo index` command like this:

```console
$ helm repo index fantastic-charts/ --url https://fantastic-charts.storage.googleapis.com
```

This will generate an updated index.yaml file and place it in the
`fantastic-charts/` directory.

## 同步本地和远程表库

Upload the contents of the directory to your GCS bucket by running
`scripts/sync-repo.sh` and pass in the local directory name and the GCS bucket
name.

For example:

```console
$ pwd
/Users/me/code/go/src/helm.sh/helm
$ scripts/sync-repo.sh fantastic-charts/ fantastic-charts
Getting ready to sync your local directory (fantastic-charts/) to a remote repository at gs://fantastic-charts
Verifying Prerequisites....
Thumbs up! Looks like you have gsutil. Let's continue.
Building synchronization state...
Starting synchronization
Would copy file://fantastic-charts/alpine-0.1.0.tgz to gs://fantastic-charts/alpine-0.1.0.tgz
Would copy file://fantastic-charts/index.yaml to gs://fantastic-charts/index.yaml
Are you sure you would like to continue with these changes?? [y/N]} y
Building synchronization state...
Starting synchronization
Copying file://fantastic-charts/alpine-0.1.0.tgz [Content-Type=application/x-tar]...
Uploading   gs://fantastic-charts/alpine-0.1.0.tgz:              740 B/740 B
Copying file://fantastic-charts/index.yaml [Content-Type=application/octet-stream]...
Uploading   gs://fantastic-charts/index.yaml:                    347 B/347 B
Congratulations your remote chart repository now matches the contents of fantastic-charts/
```

## 更新您的图表库

You'll want to keep a local copy of the contents of your chart repository or use
`gsutil rsync` to copy the contents of your remote chart repository to a local
directory.

For example:

```console
$ gsutil rsync -d -n gs://bucket-name local-dir/    # the -n flag does a dry run
Building synchronization state...
Starting synchronization
Would copy gs://bucket-name/alpine-0.1.0.tgz to file://local-dir/alpine-0.1.0.tgz
Would copy gs://bucket-name/index.yaml to file://local-dir/index.yaml

$ gsutil rsync -d gs://bucket-name local-dir/       # performs the copy actions
Building synchronization state...
Starting synchronization
Copying gs://bucket-name/alpine-0.1.0.tgz...
Downloading file://local-dir/alpine-0.1.0.tgz:                        740 B/740 B
Copying gs://bucket-name/index.yaml...
Downloading file://local-dir/index.yaml:                              346 B/346 B
```

Helpful Links:

- Documentation on [gsutil
  rsync](https://cloud.google.com/storage/docs/gsutil/commands/rsync#description)
- [The Chart Repository Guide]({{< ref "/docs/topics/chart_repository.md" >}})
- Documentation on [object versioning and concurrency
  control](https://cloud.google.com/storage/docs/gsutil/addlhelp/ObjectVersioningandConcurrencyControl#overview)
  in Google Cloud Storage
