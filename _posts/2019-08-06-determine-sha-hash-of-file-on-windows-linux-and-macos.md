---
layout: post
title: "Determine SHA hash of file on Windows, Linux, and macOS"
date: Tue, 06 Aug 2019 12:00:06 +0000
tags: command-line
# category: dev
excerpt_separator: <!--more-->
---

While it doesn’t guarantee a download hasn’t been compromised, sometimes you feel better knowing the file you downloaded matches the expected SHA hash.

Full disclosure here: while I haven’t been paid or provided anything for this blog post, I am only creating this post for me. If it helps you, great! I just keep looking this up every time I need it. Instead of wading through a bunch of Stack Overflow answers to find the exact magic command I need, I’m hoping I’ll start finding my own blog post in my search results.

<!--more-->

## SHA-256 hash commands

Here’s what I use most days. There are bound to be other ways to do this, and on more operating systems or command line shells. As well, there may be ways that are better suited for certain tasks like scripting or other automation steps. For the occasional one-off hash needs, though, they get the job done.

### macOS

```bash
shasum --algorithm 256 ~/Downloads/some-file.zip
```

### Windows (cmd.exe)

```bash
CertUtil -hashfile ~/Downloads/some-file.zip SHA256
```

### Windows (PowerShell)

Confirmed in v6.1.3 [Core], but it probably works in several earlier versions.

```powershell
Get-FileHash -Path ~/Downloads/some-file.zip -Algorithm SHA256
```

### Linux

I’ve only confirmed this on a few flavors of Ubuntu and Debian/Raspbian.

```bash
sha256sum ~/Downloads/some-file.zip
```

## Background

If you want to learn a little more about the use of hashing files, feel free to read on for some context. Otherwise, I hope these commands find us all when you need them.

### Why would I need a file hash?

As I mentioned, I usually do this to confirm a file I downloaded matches what the site said it should be when I clicked the download link. In an ideal world, this means I have downloaded the expected file exactly, and I didn’t end up downloading some compromised file or malware instead.

For example, if some software is made available from a download site to reduce their own bandwidth costs, they might build new releases to a ZIP file and upload them there for users to download. If someone were to compromise that download site and upload a malicious version of the software, unsuspecting users might click a download link and get the compromised download. If, instead, the software developers determined a cryptographic hash of the file and posted it to their own site, I could verify the same cryptographic hash of the file I downloaded from the external download site. If the hashes match, I can be reasonably sure the hash the developer generated was from the same file I downloaded.

In the real world, if someone has compromised a project, they might have compromised the project’s website as well. In that case, they could create a malicious download, determine the hash of the compromised file, and post that to the project site so that it matches the compromised file on the download site.

There is also potential for a hash to match but with different files, referred to as a hash collision. If someone could create a compromised download that had the same hash, all bets are off. The better the hashing algorithm, the less likely this can happen, but it’s something that could theoretically happen. This potential risk is why I am using SHA-256 instead of something like MD5 hashes in the above commands, because collisions are less likely in the more advanced hashing algorithm.

### When should I worry?

I don’t have a good answer here. Open source projects have been compromised in ways that would have never shown up in a file hash. If a project's source code or release pipeline is compromised, you are likely getting the hash they tell you to expect, but the files within that download are not trustworthy. There's not a lot you could do at this point, and you may not even know there is an issue.

Most of the time, I only concern myself with this when I trust a project but I don't trust the provider where the project hosts their downloads. For instance, I would trust a download from a project on GitHub more than a project that linked out to SourceForge or similar

I might also find a project where it’s hard to tell whether I’m downloading the latest release. If they provide hashes of the various versions, I can verify I have the one I wanted.

Similarly, when I'm working with legacy projects where downloads are now scattered across various download services, I may read from someone who trusts a particular download and they list the hash they found. I would possibly have more faith in a download I found with the same hash as well.

A healthy dose of paranoia is good, but it only goes so far in this realm. Stay safe out there!
