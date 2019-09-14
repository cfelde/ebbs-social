# Data formats

Content produced and published on the EBBS social platform is stored on the blockchain.

For this to be readable by the UI it needs to follow a given format. This document describes that format.

A post contains two main data areas: The post data and the post meta data.

A few other fields are included like author, timestamp, inReplyTo, and stat counters. We assume the use case for these other fields are obvious and will focus our attention on describing the post data and post meta data below.

## General data format

The post data and post meta data are stored as a sequence of bytes. The smart contract does not itself inspect this data beyond ensuring they are within the upper limits of storage constrains.

They are separated because they generally serve different roles. However, there is a possibility for overlap between them.

The content of the data is stored as a compressed JSON object, compressed using DEFLATE. Keys in the object are short to save space, and their meaning is described below. The format will also use a positive integer as a true flag, with zero being a false flag, rather than using the boolean true and false indicators, again to save on storage space.

We do not require all key/values pairs described below to be present. Only those that are needed for a given post should be included. Default values or behaviour will be used by the UI if they are missing.

Each post is mapped to a post id, which is a number starting at zero. The first post, post zero, is special and created by the forum administrator when a new forum is created. Post zero describes the forum itself.

## Post data

Post data contains the post itself, and is what the author would submit when creating a new post.

Post data structure:

{
    "1":"Post title is stored at key 1"
    "2":"Post body is stored at key 2"
    "3":"An external URL associated with the post is stored on key 3"
    "4":"An external URL associated with a post image is stored on key 4"
    "5":1
    "6":1
}

Keys 5 and 6 are used for hiding a post (UI will not display this post if value is non-zero, = true) and marking the post as NSFW, respectively.

For both of these keys the default value is zero (= false), so they are not required to be included if set to false. Note that the frontend used to view these posts is free to ignore these flags, so for example hiding a post is not the same as removing a post. Since data is stored on a blockchain, removing data is not really possible.

Realistic example:

{
    "1":"Hello world!"
    "2":"How are you all doing?"
    "3":"https://www.example.com"
    "4":"https://www.example.com/img/test.png"
}

## Post meta

Post meta contains additional content related to a post and is used by forum moderators to manage a post. For example this can be used by a moderator to mark a post as NSFW if the user didn't initially mark it as such. It can also be used in the opposite direction, for example removing a NSFW flag if the user incorrectly marked it as such.

Just as with the post data, post meta is a JSON object compressed using DEFLATE. Keys related to overall forum settings are marked with a letter rather than a number.

Post meta structure:

{
    "m":"All posts must be moderated before shown"
    "n":"All posts are to be assumed NSFW"
    "5":1
}

Realistic example:

{
    "m":1
    "n":1
}

If the above post meta is stored on the special post number zero, then all posts must first be moderated before shown (m = true) and all posts will be marked as NSFW (n = true).

Alternatively, we might have meta data stored on a regular post (not post zero), in which case this meta data overwrites the post data.

Example 1:

Post data is:

{
    "1":"Hello world!"
    "2":"How are you all doing?"
    "3":"https://www.example.com"
    "4":"https://www.example.com/img/test.png"
}

Post meta is:

{
    "5":1
}

This post is hidden because post meta sets the value of key 5 (hide flag) to true.

When parsing post data and post meta data, the UI will first take post data before applying post meta data on top. This means post meta can modify any aspect of post data.

Example:

Post data is:

{
    "1":"Hello world!"
    "2":"How are you all doing?"
    "3":"https://www.example.com"
    "4":"https://www.example.com/img/test.png"
}

Post meta is:

{
    "1":"Hello everyone!"
    "5":1
}

Combined output:

{
    "1":"Hello everyone!"
    "2":"How are you all doing?"
    "3":"https://www.example.com"
    "4":"https://www.example.com/img/test.png"
    "5":1
}

Another example:

Post data is:

{
    "1":"Hello world!"
    "2":"How are you all doing?"
    "3":"https://www.example.com"
    "4":"https://www.example.com/img/test.png"
    "5":1
}

Post meta is:

{
    "1":"Hello everyone!"
    "5":0
}

Combined output:

{
    "1":"Hello everyone!"
    "2":"How are you all doing?"
    "3":"https://www.example.com"
    "4":"https://www.example.com/img/test.png"
    "5":0
}

## Data restrictions

It is not allowed for post data to contain any keys that are not numeric. Example of invalid post data:

{
    "1":"Hello world!"
    "2":"How are you all doing?"
    "m":1
}

The m key is not allowed in post data and the front end should filter out any such data, and only allow letter based keys as part of post meta data instead.

## Others notes and considerations

Please note that the above should be followed, but we can not guarantee that post data and post meta data correctly follows this format. This is because anyone can call the smart contract APIs, also directly. So any validation must be applied also when the frontend reads content.
