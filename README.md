# Princeton Fall 2016 COS 418 Study Guide

This is a public study guide for Princeton University's Distributed Systems class.

The [syllabus](https://www.cs.princeton.edu/courses/archive/fall16/cos418/syllabus.html) describes what was covered, extracts from those presentations are used here throughout.

## Instructions for Use

Click on the unit you want to review. Read.

## Instructions for Adding Content

Useful markdown [cheat sheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).

0. Fork this repo
0. Create a folder for the unit you want to add (follow the others as an example - use hyphens for spaces and all lowercase).
0. Add a single file in that folder, `README.md`, following the same format as the others. In particular, it should look like the excerpt below. Auxiliary materials are welcome in that unit's folder.
0. To preview your changes (before adding them), use [grip](https://github.com/joeyespo/grip).
0. Create a pull request, I'll merge it in.

```markdown
# Summary

Key points are tersely listed here.

* use bullets

Other requirements:

0. **Someone who understands everything here should be prepared for the exam.**
0. This should be a medium-length outline.

# Detailed Explanation

This section is optional. 

It contains in-depth descriptions of the content described above. 

The various nuances described here are only to help understanding of the topic.
```

## What's not done yet:

primary-backup

spark

rpc

2pc

raft

chord

paxos failure examples

cops

dynamo

sporc (fork* definition, OT definition)

transactions: 2PL Permits Interleaved Access

transactions: 2PL Doesn't Exploit Full Concurrency

transactions: Undisciplined Locking causes Non-serializable Schedules

transactions: ARIES Data Structures

transactions: ARIES Crash Recovery Trace

transactions: Black/White Marble Example

crypto

bitcoin (inc. blockchain)

graph processing

CDN

roofnet

cluster scheduling

stream scheduling
