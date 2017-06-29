# 0010: External file handling

- **Author:** Chandra Lash
- **Status:** WIP Draft Proposal, 2017-07-11
- **Backlog Issue:** [#1646](https://github.com/Cadasta/cadasta-platform/issues/1646)


## Problem overview

The Cadasta Platform has inconsistent uses of referencing external files in the Platform. Not only does this complicate the code, it could potentially impede performance and create bugs over time due to uncontrollable third-party changes.


## Suggested solution

- Cadasta should adopt a standard practice of hosting all files locally on our servers.


## Other considerations

- Exceptions should be made on a case-by-case basis only. For example our pricing package with Inline Manual prevents us from using a standalone player.
