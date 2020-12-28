# High-availability plugin: delta reindex

## When

In an HA setup, the necessity of reindexing projects on all the nodes of the cluster
can arise, for example, when upgrading Gerrit version or when recovering from
indexes misalignment.

## Why

Indexing operation might take long time (hours) for a big Gerrit installation.
Reindexing all the nodes of the cluster might end up in taking days. The reindexing
operation is resource intense, hence less resources are available for the normal
activities during reindexing. Minimising indexing time becomes crucial.

## How

Let’s take the example of a Gerrit cluster with two nodes, where indexes of `node2`
are not in sync with `node1`. The quickest way to align them is a **delta reindex**,
as follow:
* Put `node1` in readonly mode
* Compress the index directory on `node1` (`tar -cvfz index.tar.gz <gerrit_home>/index`)
* Take note of the current time, le's call it *startTime*
* Put `node1` in readwrite mode
* Copy *index.tar.gz* to `node2`
* Stop Gerrit on `node2`
* Backup the index directory on `node2` (`cp -R <gerrit_home>/index <gerrit_home>/index_backup`)
* Uncompress the index archive on `node2` (`tar -xvf index.tar.gz <gerrit_home>/index`)
* Move back the date of the [last index update timestamps](https://gerrit.googlesource.com/plugins/high-availability/+/refs/heads/stable-3.1/src/main/resources/Documentation/about.md#last-index-update-timestamp-storage)
to "*startTime* - 10min" with this format `yyyy-mm-ddTHH:MM:SS.ss`
* Restart Gerrit on `node2`

In this case, not all the changes will be reindexed, but only the changes with
date greater than *startTime*, hence the name **delta reindex**.

At this point the indexes on the two nodes are aligned.
