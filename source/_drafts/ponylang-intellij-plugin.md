title: "ponylang-intellij-plugin"
tags:
- pony lang
- intellij
categories:
- Software Development
- Intellij
---

I'm a Java/Scala developer and I am used to Intellij Idea for my work. When I started my interest in Pony, I used Sublime Text for a short while and then switched to Atom for code highlighting. But what I miss
in both these tools are all the information that Intellij provides - code completion, inspection, refactoring, analysis.

I decided to start my own Intellij plugin to support Pony language on that platform. There is a nice tutorial on adding new language support for Intellij in [this blog post](http://blog.jetbrains.com/idea/2013/01/how-to-write-custom-language-support-plugins/)
so I started with the basics and hit a wall. This tutorial mentions using the GrammarKit plugin which generates a lot of boilerplate from BNF grammars. The issue I had was that the grammar file
in Pony repository was Antlr v3 grammar and I spent a few days investigating how to convert one to another. After some unsuccessful attempts I had to postpone work on this little project.

Today I was playing with Pony and noticed this small line in the output of `ponyc --help` - `--bnf Print out the Pony grammar as human readable BNF.` I was shocked - I don't know when this was implemented (current Pony version is 0.1.7)
but it saves the plugin development! It still wasn't the format expected by GrammarKit plugin but it was very easy to adapt. After a few moments I had a grammar file with generated tokens and all stuff required by the parser!

That's just one small step towards the first version of the plugin. The features I'd like to have in the very first published version are: syntax highlighting, pony files support, run configuration to build the project.
This would allow people to start developing Pony applications from within Intellij. There's still a lot of work to do, so any help is appreciated. The project is located in [this GitHub repository](https://github.com/pbuda/ponylang-intellij-plugin). Fork away!
