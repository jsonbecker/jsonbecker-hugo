---
title: The Slow Trek to Pelican
date: 2012-12-20
author: Jason
category: Uncategorized
tags: code, pelican, python, regex, wordpress
---

*Update: Please see below for two solutions.*

I have grown increasingly unhappy with Wordpress lately. My blog is simple. My design tastes are simple. My needs are simple. I like control. I am a geek. And I really need an excuse to learn Python, which seems to be rapidly growing into one of the most important programming languages for a data analyst.

I have decided to migrate this blog over to [Pelican][], a static site generator written in Python. Static sites are the "classic" way to do a webpage-- just upload a bunch of HTML and CSS files, maybe some Javascript. But no databases and no constructing the page a user sees in the browser as they request it. This puts substantially less strain on a web server and makes it far easier to export and move a webpage since all you need to do is duplicate files. What makes static sites a real pain is that there is a lot of repetition. Folks adopted dynamic sites that use content management system so that they can write a page called "post.php" one time, and for each unique post just query a database for the unique content. The frame around the post, layout, components, etc are all just written once. Static site generators allow you to build a webpage using a similar, but far more stripped down, layout system. However, rather than generate each page on the web server, you generate each page by running a script locally that transforms plain text documents into well-formed HTML/CSS. Then you can just upload a directory and the whole site is ready to go.

Pelican comes with a pretty good script that will take Wordpress XML that's available via the built-in export tools and transform each post into a [reStructuredText][] files, a format similar to [Markdown][]. I prefer Markdown so I used [pandoc][] to convert all my \*.rst posts into \*.md files.

So far, so good.

But one of the really big problems I had with Wordpress was a growing dependency on plugins that added non-standard, text-based markup in my posts that would be rendered a particular way. For example, text surrounded by two parenthesis, '[^0]', became a footnote. For code syntax highlighting, I use a "short code", which puts "sourcecode language='r'", for example, between brackets []. All of these plugins have been great, but now when you try to export a post you get the non-standard markup in-line as part of your posts. It makes it very difficult to recreate a post the way it looks today.

This presents a great opportunity to learn a little Python. So I have begun to scrounge together some basic Python knowledge to write some scripts to clean up my Markdown files and convert the syntax of the short codes that I have used to properly formatted Markdown so that when I run the pelican script it will accurately reproduce each post.

Unfortunately, I've hit a snag with my very first attempt. Footnotes are a big deal to me and have standard Markdown interpretation. In Markdown, footnotes are inserted in the text where "[\^\#]" appears in the text, where \# = the footnote identifier/key. Then, at the end of the document, surrounded by new lines, the footnote text is found with "[\^\#]: footnote text" where \# is the same identifier. So I needed to write a script that found each instance of text surrounded by two parentheses, insert the [\^\#] part in place of the footnote, and then add the footnote at the bottom of the post in the right format.

I created a test text file:

    :::
    This is a test ((test footnote)).
    And here is another test ((footnote2)). Why not add a third? ((Three
    Three)).

The goal was to end up with a file like this:

    :::
    This is a test [^1]. And here is another
    test [^2]. Why not add a third? [^3].

    [^1]: test footnote

    [^2]: footnote2

    [^3]: Three Three

Unfortunately, the output isn't quite right. My best attempt resulted in
a file like this:

    :::
    This is a test [^1] And here is another te[^2])). Why not add a
    t[^3]ree)).

    [^1]: ((test footnote))

    [^2]: ((footnote2))

    [^3]: ((Three Three))


Ugh.

So I am turning to the tiny slice of my readership that might actually know Python or just code in general to help me out. Where did I screw up? The source to my Python script is below so feel free to comment here or on this [Gist][]. I am particularly frustrate that the regex appears to be capturing the parenthesis, because that's not how the same code behaves on [PythonRegex.com][].

If anyone can help me with the next step, which will be creating arguments that will understand an input like *.rst and set the output to creating a file that's *.md, that would be appreciated as well. 

    :::python
    import re
    
    p = re.compile("\(\(([^\(\(\)\)]+)\)\)")
    file_path = str(raw_input('File Name >'))
    text = open(file_path).read()
    
    footnoteMatches = p.finditer(text)
    
    coordinates = []
    footnotes = []
    
    # Print span of matches
    for match in footnoteMatches:
        coordinates.append(match.span())
        footnotes.append(match.group())
    
    for i in range(0,len(coordinates)):
        text = (text[0:coordinates[i][0]] + '[^' + str(i+1)+ ']' +
                text[coordinates[i][1]+1:])
        shift = coordinates[i][1] - coordinates[i][0]
        j = i + 1
        while j < len(coordinates):
            coordinates[j] = (coordinates[j][0] - shift, coordinates[j][1] - shift)
            j += 1
    
    referenceLinkList = [text 1="'
    '" language=","][/text]
    for i in range(0, len(footnotes)):
        insertList = ''.join(['\n', '[^', str(i+1), ']: ', footnotes[i], '\n'])
        referenceLinkList.append(insertList)
    
    text = ''.join(referenceLinkList)
    
    newFile = open(file_path, 'w')
    newFile.truncate()
    newFile.write(text)
    newFile.close()

## Update with solutions:

I am happy to report I now have two working solutions. The first one comes courtesy of [James Blanding][] who was kind enough to [fork][] the gist I put up. While I was hoping to take a look tonight at his fork tonight, [Github was experiencing some downtime][].  So I ended up fixing the script myself a slightly different way (seen below). I think James's approach is superior for a few reasons, not the least of which was avoiding the ugly if/elif/else found in my code by using a global counter. He also used .format() a lot better than I did, which I didn't know existed until I found it tonight.

I made two other changes before coming to my solution. First, I realized my regex was completely wrong. I didn't want to capture anything within the two parenthesis when no parenthesis were contained, as the original regex did. Instead, I wanted to make sure to preserve any parenthetical comments contained within my footnotes. So the resulting regex looks a bit different. I also switched from using user input to taking in the filepath as an argument.

My next step will be to learn a bit more about the os module which seems to contain what I need so that this Python script can behave like a good Unix script and know what to do with one file or a list of files as a parameter (and of course, most importantly, a list generated from a wild card like \*.rst). I will also be incorporating the bits of James's code that I feel confident I understand and that I like better.

Without further ado, my solution (I updated the gist as well):

    :::python
    from sys import argv
    import re
    
    name, file_path = argv
    
    p = re.compile(r"[\s]\(\((.*?[)]{0,1})\)\)[\s]{0,1}")
    # The tricky part here is to match all text between "((""))", including as 
    # many as one set of (), which may even terminate ))). The {0,1} captures as
    # many as one ). The trailing space is there because I often surrounded the 
    # "((""))" with a space to make it clear in the WordPress editor.
    
    # file_path = str(raw_input('File Name >'))
    text = open(file_path).read()
    
    footnoteMatches = p.finditer(text)
    
    coordinates = []
    footnotes = []
    
    # Print span of matches
    for match in footnoteMatches:
        coordinates.append(match.span())
    # Capture only group(1) so you get the content of the footnote, not the 
    # whole pattern which includes the parenthesis delimiter.
        footnotes.append(match.group(1))
    
    newText = []
    for i in range(0, len(coordinates)):
        if i == 0:
            newText.append(''.join(text[:coordinates[i][0]] +
                                   ' [^{}]').format(i + 1))
        elif i < len(coordinates) - 1 :
            newText.append(''.join(text[coordinates[i-1][1]:coordinates[i][0]] +
                              ' [^{}]').format(i + 1))
        else:
            newText.append(''.join(text[coordinates[i-1][1]:coordinates[i][0]] +
                              ' [^{}]').format(i + 1))
            # Accounts for text after the last footnote which only runs once.
            newText.append(text[coordinates[i][1]:]+'\n')
    
    endNotes = []
    for j in range(0, len(footnotes)):
        insertList = ''.join(['\n','[^{}]: ', footnotes[j], '\n']).format(j + 1)
        endNotes.append(insertList)
    
    newText = ''.join(newText) + '\n' + ''.join(endNotes)
    
    newFile = open(file_path, 'w')
    newFile.truncate()
    newFile.write(newText)
    newFile.close()


[Pelican]: http://docs.getpelican.com/en/3.1.1/
[reStructuredText]: http://docutils.sourceforge.net/rst.html
[Markdown]: http://daringfireball.net/projects/markdown/
[pandoc]: http://johnmacfarlane.net/pandoc/
[Gist]: https://gist.github.com/4342554#file-wpfootnotestomarkdown-py
[PythonRegex.com]: http://www.pythonregex.com
[James Blanding]: https://github.com/ilikepi
[fork]: https://gist.github.com/4355865
[Github was experiencing some downtime]: http://news.ycombinator.com/item?id=4957935

