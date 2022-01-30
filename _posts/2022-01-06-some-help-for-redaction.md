---
title: Some help for redaction
date: 2021-12-25 23:53:00 +0800
categories: ["2022", moderation]
tags: [post,tutorial,jekyll]
author:
  name: Club Robot
  link: https://github.com/ClubRobotInsat
image:
  src: https://image.freepik.com/free-vector/men-with-computer-technology-documents-strategy_24877-53513.jpg
  width: 500   # in pixels
  height: 200   # in pixels
  alt: Welcome to redaction guide
pin: true
render_with_liquid: false
comments: false
math: false
mermaid: true
---
In this post you will find all required information for posting in this website

## Markdown (.md) redaction
At first you have to take a look on markdown syntax [here](https://www.markdownguide.org/basic-syntax/)

## Image example
If you need place some images in your posts. Use the following syntax.

```yaml
![Alt word if image isn't loaded](/assets/img/favicons/android-chrome-512x512.png){: width="200" height="100"}
*Image description phrase*
```
For each image the width and the height are in pixels.
<br>You can save your images on /assets/img/posts or just use the url instead (Ex. https://ursd.org/wp-content/uploads/2020/12/1.png).
### Result :
![Alternative (descriptive) word if image isn't loaded](/assets/img/favicons/android-chrome-512x512.png){: width="300" height="200"}
*Few words about image*

## Code block
Jekyll supports several types of code blocks
### For any code you can use
````
```
code
```
````
*Result*
```
Literally any code
```
### There is a possibility to specify the programming language
````
```rust
code
```
````
*Result*
```rust
!print("Hello world!")
```
## About post configuration

You can find a template for posts at main branch named *post_template.md*
<br>There are some details to add.
```yaml
title: The main title
date:   dd/mm/yy hh:mm:ss +0800
categories: ["year", category2]     # Up to 2 categories
tags: [tag1,tag2,tag3]              # You can add as much tags as you wish (but dont abuse)
author:
  name: Your name
  link: https://github.com/YourGithubUsername
image:
  src: image url
  width:                            # in pixels
  height:                           # in pixels
  alt: alternative text             # An alternative text which will appear if the image isn't loaded
```
