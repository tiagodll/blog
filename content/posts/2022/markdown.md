---
title: "Markdown - the basics"
date: 2022-04-22T18:00:00+02:00
draft: false
tags: ['Markdown']
featured_image: https://en.wikipedia.org/wiki/Markdown#/media/File:Markdown-mark.svg
description: Markdown is slowly turning into the defaul text markup for the internet. Why is that?
---

# Background

Why I am writing about markdown, isn't this boring?

No my little padawan, markdown have quickly climbed to be my favorite text editing tool.

During a few years I worked in a custom blog engine. After that in a social network for traders. 

In both cases, editing the content was crucial for the software, and having tried many, *many* different WYSIWYG editors, I gave up.

No matter what we did (yes, specially in the second case, there was a big team involved), the users always found ways to break our editor and generate invalid data. 

This was almost always caused by copying and pasting text from Microsoft word, or some website.

Fast forward some years, I created an only editor for interactive books. 

Again, WYSIWYG editors were not cutting it. And then I gave a try to markdown. Simple, easy, clean and easily editable text. 

Couldn't be happier.

# Where can you use it

- [wikipedia](https://www.wikipedia.com/)
- [github](https://www.github.com/)
- [stackoverflow](https://www.stackoverflow.com/)
- [Redmine](https://www.redmine.org/)
- [Reddit](https://www.reddit.com)
- [Jira](https://www.atlassian.com/software/jira)
- [Joplin](https://joplinapp.org/) - nice todo list, and note taking app
- [Obsidian](https://obsidian.md/) - where I write this blogpost
- [Hugo](https://gohugo.io/) - blog engine I use in this blog
- so many more...

# How to use it

## Titles
\# : biggest title (H1)

\#\# : smaller (H2)

\#\#\# : smaller (H3)
...

## Text formatting
\*\* your text \*\* : bold

\* your text \*: italic

\~\~text\~\~: strikethrough

\=\=text\=\=: highlighted

\*OBS: To have a line break you must hit enter twice. Because reasons...

## Links
\!\[ title \]\( url \): image

\[ title \]\( url \): link

## Lists
\- : list item 

1\. : ordered list

\- \[ \] : check item

\- \[x\] : checked check item

## Syntax highlighting
\`\`\`language

your code
\`\`\`
ex:
```haskell
getNotes :: String -> HTTPure.ResponseM
getNotes token = do
  notes <- DB.getNotes token
  HTTPure.ok $ JSON.writeJSON (M.Response notes)
```

### Tables:
```
First Header | Second Header
------------ | ------------
First Column | Second Column
Another row  | Second Column
```
 
ex:
First Header | Second Header
------------ | ------------
First Column | Second Column
Another row  | Second Column


# Conclusion
I hope this helped you see how powerful markdown is, how it can help fulfill all the needs for most cases (I know, having colors would be nice, but are they **needed**?)

Hopefully it was already enough to get your attention to this topic.

There are [much more detailed guides online](https://www.w3schools.io/file/markdown-introduction/), with specific tags for each product (yes, basic markdown is supported by all, but many apps introduce extra options).

Happy text editing!
