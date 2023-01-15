---
layout: posts
title:  "Writing about code"
date:   2023-01-14 15:02:59 -0800
categories: 
    - code
    - powershell
    - writing
---

In the long run, one of the things I will likely write most about is scripting and coding wih PowerShell and Python. Alongside this effort, I am in the process of updating most of my scripts and adding them to public repositories to share. I learned a long time ago that the best way to know something is to:

+ Learn it
+ Do it
+ Teach (or share) it

Sharing things I have learned has probably  taught me as much about them as discovering and using them

## Today's topic: HashSets

I have often used hashtables to filter out duplicate values. For example if we start with the following strings:

```pwsh
$thingsTheySaid = @(
    "His first name is Johnny."
    "Johnny's last name is Doe."
    "Johnny is not an idiot."
    "Tess' favorite color is purple, but does'nt like green!"
    "'Tess' loves Johnny's cooking!"
)
```

Create some RegEx to just get words

```pwsh
$wordRgx = @"
^\'(?<aWord>.+)\'$|^\"(?<aWord>.+)\"$|^(?<aWord>[A-Za-z]+(\'[A-Za-z]*)?).*$
"@
```

The do some splitting and matching before dropping the words in a hashtable we can get a list of every unique word along with how often they appear with something like

```pwsh
$uniqueWordHashtable = @{}

foreach ($thing in ($thingsTheySaid -split (" "))) {
    if ($thing -match $wordRgx) {
        if ($uniqueWordHashtable["$($Matches["aWord"])"]) {
            $uniqueWordHashtable["$($Matches["aWord"])"]++
        } else {
            $uniqueWordHashtable["$($Matches["aWord"])"] = 1
        }
    }
}

$uniqueWordHashtable
```

That's all well and good when additional information (like how often the words appear) can be useful but sometimes I only need to know if data appears and do not need any more information. I used to do something like:

```pwsh
$uniqueWordHashtable["$($Matches["aWord"])"] = [string]::empy
```

Or sometimes used `$true`, `$false`, or `0` just to have some data to match that key. Not awful when I'm first getting the data but extra work if I already had the data, such as if I'd done:

```pwsh
$allWords = foreach ($thing in ($thingsTheySaid -split (" "))) {
    if ($thing -match $wordRgx) {
        "$($Matches["aWord"])"
    }
}
```

Giving me data including duplicates. In order to get unique values I either have to rewrite my code to use a hashtable or use an alternate solution, like a .Net [hashset](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.hashset-1?view=net-7.0). Going the hashteble route I would have to loop through the `$allWords` string array to try to add key/value pairs. I could do something similar with a hashset if I really wanted to or I could just cast it as a string hashset:

```pwsh
[System.Collections.Generic.HashSet[string]]$allWords
```

or, if I wanted to play it safe

```pwsh
$allWords -as [System.Collections.Generic.HashSet[string]]
```

I'm partial to the casting route because you never know when all the words, including suplicates, may prove useful. No code is ever truly wasted.

All put together it would look something like:

```pwsh
$wordRgx = @"
^\'(?<thisWord>.+)\'$|^\"(?<thisWord>.+)\"$|^(?<thisWord>[A-Za-z]+(\'[A-Za-z]*)?).*$
"@

$thingsTheySaid = @(
    "His first name is Johnny."
    "Johnny's last name is Doe."
    "Johnny is not an idiot."
    "Tess' favorite color is purple, but does'nt like green!"
    "'Tess' loves Johnny's cooking!"
)

$allWords = foreach ($thing in ($thingsTheySaid -split (" "))) {
    if ($thing -match $wordRgx) {
        "$($Matches["thisWord"])"
    }
}

$allWords -as [System.Collections.Generic.HashSet[string]]
```

## In closing

Writing exercise over. Much lived evrything else I've been writing, it took much longer than I liked, but it didn't turn out too bad. It should get easier with practice.

As alway, keep playing, exploring, and learning...and of course, dont forget to enjoy the day.