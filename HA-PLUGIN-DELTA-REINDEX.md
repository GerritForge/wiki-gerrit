# High-availability plugin: delta reindex

## When

In an HA setup, the necessity of reindexing projects on all the nodes of the cluster
can arise, for example, when upgrading Gerrit version or when recovering from
indexes miss-alignment.

## Why

Indexing operation might take long time (hours) for big Gerrit installation.
Reindexing all the nodes of the cluster might end up in taking days. Reindexing
operation is resource intense, hence less resources are available for the normal
activities during reindexing. Minimising indexing time becomes crucial.

## How

Let’s take the example of a Gerrit cluster with 2 nodes, where indexes of `node2`
are not in sync with `node1`. The quickest way to align them is a **delta reindex**,
as follow:
* Put `node1` in readonly mode
* Compress *<gerrit_home>/index* directory on `node1` into *index.tar.gz*
* Copy *index.tar.gz* to `node2`
* Stop Gerrit on `node2`
* Uncompress *index.tar.gz* *<gerrit_home>/index* in `node2`
* Move back the date of the following files to a time before `node1` was put in
readonly (let's call it *startTime*):
    * *<gerrit_home>/data/high-availability/group*
    * *<gerrit_home>/data/high-availability/account*
    * *<gerrit_home>/data/high-availability/change*
* Restart Gerrit on `node2`

In this case, not all the changes will be reindexed, but only the changes with
date greater than *startTime*, hence the name **delta reindex**.

At this point the indexes on the 2 nodes are aligned.
