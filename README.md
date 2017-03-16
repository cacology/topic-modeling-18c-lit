# Topic Modeling Eighteenth-Century Literature

Prepared for Professor Taylor Walle's English 335: Radical Jane, but
you might find it useful too.  Topic modeling is particularly easy for
eighteenth-century canonical works because decent text files almost
always exist.  It draws heavily on
this
[tutorial](http://programminghistorian.org/lessons/topic-modeling-and-mallet) at
the Programming Historian.  It's excellent and you may find it more
useful than this one.  It also assumes you're using a Mac or another
Unix-based operating system.  POSIX provides a wide range of extremely
useful tools for manipulating plain text, but I'm sure you could use
another platform; I just can't tell you how to do it.

The first thing to do is to find the text or texts that
you want to work with.  There are a couple of great sources:

[The Gutenberg Project](http://www.gutenberg.org)

: The Gutenberg project is old and their texts often have encoding
issues, but they have a lot more than just eighteenth-century
materials, so are worth getting to know.

[ECCO-TCP](http://quod.lib.umich.edu/e/ecco/)

: The Eighteenth Century Collections Online: Text Creation Partnership
is actively developing transcriptions from the digital reproductions
in ECCO

[TypeWright](http://www.18thconnect.org/typewright/documents)

: TypeWright is a project of 18thConnect which provides not only the
text from ECCO-TCP, but a mechanism to edit and improve texts.
They provide a mechanism for peer review, so you can pick any text and
type it up yourself.

For this example, we'll just use The Gutenberg Project, since they're
easy to get and usually clean, if strangely encoded.  Suppose you
download Jane Austen's
novel [Emma](http://www.gutenberg.org/ebooks/158).  The best format to
work with is the "Plain Text UTF-8".
Download that [one](http://www.gutenberg.org/files/158/158-0.txt), or
get it from this repository.

You'll need to use the command line, so if you don't know it, there's
a good
[tutorial](http://programminghistorian.org/lessons/intro-to-bash) at
The Programming Historian.  Open a terminal and navigate to the
directory with the file in it and run:

    file 158-0.txt

You'll get a response like:

    158-0.txt: UTF-8 Unicode (with BOM) text, with CRLF line terminators

This tells us the file is in UTF-8, which is a good sign, but has
a little block at the beginning, a BOM, that signals that.  Not really
a problem for us, but since we'll be using a Unix command line the
"CRLF line terminators" will be.  Let's clear them out by running the
"translate characters" command `tr`:

    tr -d '\r' < 158-0.txt > Emma.txt

The `-d` flag tells `tr` to merely delete all the carriage returns.
These used to be part of how a text file was described for DOS and
other systems.  The CR, carriage return, would return the cursor to
the beginning of the line, while the LF, line feed, would scroll
another line down.  This made more sense when terminals were actually
printers and those were two separate actions.  Anyhow, we should now
have a clean `Emma.txt` file.  Let's take a look at it:

    less Emma.txt

You'll note, and this is important, that the chapters are designated
with the word "CHAPTER" in all capital letters.  This will be useful
later, but you'll also note a bunch of text at the beginning and end.
Open the file in your favorite text editor and delete all of
that noise.  I use Emacs, but any plain text editor will work.

Now, we want to divide the text into the units that we want to
consider a topic.  For now, I'll do chapters, since they seem to be
about one thing, but you could easily do sentences or paragraphs.
We'll be using `csplit` which knows how to divide a text by
matching patterns.  This command:

    csplit -f EmmaChap -k -n 3 Emma.txt /CHAPTER/ {999}

Tells `csplit` to cut up Emma.txt into files name EmmaChap with '3'
digits after them (so they can be numbered) and to `-k`eep (i.e. keep)
the files it generates even if it encounters an error, like no match.
The `{999}` at the end says to match nine-hundred and ninety-nine
times, which works because we can have EmmaChap000 through
EmmaChap999, though we won't need that many.  If you decided to cut it
into paragraphs, you might need `-n 4` and `{9999}` and for a longer
text, `-n 6` and `{999999}` wouldn't be crazy.  The last bit
`/CHAPTER/` tells `csplit` to make a new file every time it encounters
the word "CHAPTER" in all caps.  (If you want to understand what's
going on with this, read about "basic regular expressions"; better
yet, you could learn about Perl or something that has more advanced
regular expressions; Google them)  If you run that command, type `ls`
and you should see something like:

    158-0.txt   EmmaChap008 EmmaChap018 EmmaChap028 EmmaChap038 EmmaChap048
    Emma.txt    EmmaChap009 EmmaChap019 EmmaChap029 EmmaChap039 EmmaChap049
    EmmaChap000 EmmaChap010 EmmaChap020 EmmaChap030 EmmaChap040 EmmaChap050
    EmmaChap001 EmmaChap011 EmmaChap021 EmmaChap031 EmmaChap041 EmmaChap051
    EmmaChap002 EmmaChap012 EmmaChap022 EmmaChap032 EmmaChap042 EmmaChap052
    EmmaChap003 EmmaChap013 EmmaChap023 EmmaChap033 EmmaChap043 EmmaChap053
    EmmaChap004 EmmaChap014 EmmaChap024 EmmaChap034 EmmaChap044 EmmaChap054
    EmmaChap005 EmmaChap015 EmmaChap025 EmmaChap035 EmmaChap045 EmmaChap055
    EmmaChap006 EmmaChap016 EmmaChap026 EmmaChap036 EmmaChap046
    EmmaChap007 EmmaChap017 EmmaChap027 EmmaChap037 EmmaChap047

We now have Emma divided into 000 through 055 chapters, that is
fifty-six chapters counting from zero.  Put all those chapters into
one directory thus:

    mkdir EmmaChaps
    mv EmmaChap0* EmmaChaps

Now, we're ready to get into Mallet, but first you need to
install it.  It's really easy, simply follow the
directions [here](http://mallet.cs.umass.edu/download.php) and
remember where you put it.  For sake of the exercise, let's say we put
it in `mallet-2.0.8` in the same directory as these files.  That's not
a great place to leave it, but good enough for now.  The first thing
you need to do is to convert these chapters into a Mallet file, with
the stopwords removed and the sequence preserved.  Mallet is a weird
tool, but well-documented.  Type:

    ./mallet-2.0.8/bin/mallet import-dir

And you'll get a long list of options.  Look over them, but there are
only a few we need right now.  Mallet can do a lot more than
topic-modeling, but what we need is only a few options.

    ./mallet-2.0.8/bin/mallet import-dir --input EmmaChaps --output Emma.mallet --remove-stopwords --keep-sequence

Stopwords are really common English words that muck things up, like
"the" and "a".  Most of what we read and write have these in them so
they can be a problem for doing some topic-modeling, then again, you
could leave them in and see what happens.  The flag
`--remove-stopwords` tells Mallet to take them out.  The flag
`--keep-sequence` is needed to preserve the structure of the texts
which the topic modeler needs.  (Though, something like a classifier
doesn't want, but that's another topic.)  The flags `--input` and
`--output` specify the directory going in and the file going out.
Run the command and you should have Emma.mallet read to use.  Now we
can train a topic modeler on it.

Understanding *what* exactly a topic model does takes some time.
Refer to the Programming Historian post above, which has a very good
list of resources, once you've gone through this, but for now, let's
just do it blindly.  You need to call Mallet again, but this time to
train the topic model.  Type this to see the options:

    ./mallet-2.0.8/bin/mallet train-topics

You'll see a bunch of options.  They're all documented, more or less,
in a easy Google search or
on
[http://mallet.cs.umass.edu/topics.php](http://mallet.cs.umass.edu/topics.php) so
play with them later, but for now, type this:

    ./mallet-2.0.8/bin/mallet train-topics --input Emma.mallet --num-topics 10 --output-topic-keys keys.tsv --output-doc-topics doc-topics.tsv --optimize-interval 10

The flag `--num-topics` gives the number of topics to model.  The "keys" that are output are the first few words associated with each topic and the "doc-topics" are the topics associated with each topic.  The flag `--optimize-interval` tweaks the training program to make the topics fit a bit better.  Programming Historian suggests 20, the Mallet page suggests 10; it's all voodoo unless you spend more time studying the algorithm.  That's it!  You've done it!  Now open the files up and take a look at the topic you've generated.  A word on the stream of text that comes out when you run that command, the `LL/token` value is how well the topics fit the text.  A higher number is better, but notice that they're *negative* so when it runs it will probably go from somewhere around -9 to -8.523.  Among other things, you could fiddle with optimization intervals and the number of topics to make a better fit.

For extra fun, try generating other numbers of topics or cutting
a text up another way.  This command will split a text by paragraphs:

    csplit -f EmmaPar -n 6 -k Emma.txt /^.$/ {999999}

If you research basic regular expressions this will make sense.
It finds the space between paragraphs, but it gives you lots
of files.  Repeat the above to create another Mallet file and you can
model that way.
