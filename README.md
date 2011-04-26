BibTeX-Ruby
===========

The BibTeX-Ruby package contains a parser for BibTeX bibliography files and a
class structure to manage, search, or convert BibTeX objects in Ruby. It is
designed to support all BibTeX objects (including @comment,
string-replacements via @string, as well as string concatenation using '#')
and handles all content outside of BibTeX objects as 'meta comments' which may
or may not be included in post-processing.


Quickstart
----------

    $ [sudo] gem install bibtex-ruby
    $ irb
    >> require 'bibtex'
    => true
    > bib = BibTeX.open('./ruby.bib')
    => book{pickaxe,
      address = {Raleigh, North Carolina},
      author  = {Thomas, Dave, and Fowler, Chad, and Hunt, Andy},
      date-added = {2010-08-05 09:54:07 0200},
      date-modified = {2010-08-05 10:07:01 0200},
      keywords = {ruby},
      publisher = {The Pragmatic Bookshelf},
      series = {The Facets of Ruby},
      title = {Programming Ruby 1.9: The Pragmatic Programmers Guide},
      year = {2009}
    }
    >> bib[:pickaxe].year
    => "2009"
    >> bib[:pickaxe][:title]
    => "Programming Ruby 1.9: The Pragmatic Programmer's Guide"
    >> bib[:pickaxe].author = 'Thomas, D., Fowler, C., and Hunt, A.'
    => "Thomas, D., and Fowler, C., and Hunt, A."
    >> bib['@book'].length
    => 1
    >> bib['@article'].length
    => 0
    >> bib['@book[year=2009]'].length
    => 1
    >> bib['@book[year=2011]'].length
    => 0


Installation
------------

If you just want to use it:

    $ [sudo] gem install bibtex-ruby

If you want to work with the sources:

    $ git clone http://github.com/inukshuk/bibtex-ruby.git
    $ cd bibtex-ruby
    $ [sudo] bundle install
    $ rake racc
    $ rake rdoc
    $ rake features
    $ rake test

Or, alternatively, fork the [project on GitHub](http://github.com/inukshuk/bibtex-ruby.git).


Requirements
------------

* The parser generator [racc](http://i.loveruby.net/en/projects/racc/) is
  required to generate parser.
* The **json** gem is required on older Ruby versions for JSON export.

The bibtex-ruby gem has been tested on Ruby versions 1.8.7 and 1.9.2; it has
been confirmed to work with REE 1.8.7 x86_64 and JRuby 1.5.6 x86_64-java;
however, there have been some [issues](https://github.com/inukshuk/bibtex-ruby/issues)
with MacRuby implementations.



Usage
-----

It is very easy to use BibTeX-Ruby. You can use the top level utility methods
**BibTeX.open** and **BibTeX.parse** to open a '.bib' file or to parse a string
containing BibTeX contents. Normally, BibTeX-Ruby will discard all text outside
of regular BibTeX elements; however, if you wish to include everything, simply add
`:include => [:meta_comments]` to your invocation of **BibTeX.open** or **BibTeX.parse**.

Once BibTeX-Ruby has parsed your '.bib' file, you can easily access individual entries.
For example, if you set up your bibliography as follows:

    bib = BibTeX.parse <<-END
    @book{pickaxe,
      address = {Raleigh, North Carolina},
      author = {Thomas, Dave, and Fowler, Chad, and Hunt, Andy},
      date-added = {2010-08-05 09:54:07 +0200},
      date-modified = {2010-08-05 10:07:01 +0200},
      keywords = {ruby},
      publisher = {The Pragmatic Bookshelf},
      series = {The Facets of Ruby},
      title = {Programming Ruby 1.9: The Pragmatic Programmer's Guide},
      year = {2009}
    }
    END
    
You could easily access it, using the entry's key, 'pickaxe', like so: `bib[:pickaxe]`;
you also have easy access to individual fields, for example: `bib[:pickaxe][:author]`.
Alternatively, BibTeX-Ruby accepts ghost methods to conveniently access an entry's fields,
similar to **ActiveRecord::Base**. Therefore, it is equally possible to access the
'author' field above as `bib[:pickaxe].author`.

Instead of parsing strings you can also create BibTeX elements directly in Ruby:

    > bib = BibTeX::Bibliography.new
    > bib << BibTeX::Entry.new({
    >   :type => :book,
    >   :key => 'rails',
    >   :address => 'Raleigh, North Carolina',
    >   :author => 'Ruby, Sam, and Thomas, Dave, and Hansson, David Heinemeier',
    >   :booktitle => 'Agile Web Development with Rails',
    >   :edition => 'third',
    >   :keywords => 'ruby, rails',
    >   :publisher => 'The Pragmatic Bookshelf',
    >   :series => 'The Facets of Ruby',
    >   :title => 'Agile Web Development with Rails',
    >   :year => '2009'
    > })
    > book = BibTeX::Entry.new
    > book.type = :book
    > book.key = 'mybook'
    > bib << book


### Queries

Since version 1.3 BibTeX-Ruby implements a simple query language to search
Bibliographies. For instance:

    >> bib[:key]
    => Returns entries with key 'key'
    >> bib['key']
    => Same as above
    >> bib['@article']
    => Returns all entries of type 'article'
    >> bib['@preamble']
    => Returns all preamble objects (this is the same as Bibliography#preambles)
    >> bib[/ruby/]
    => Returns all objects that match 'ruby' anywhere
    >> bib['@book[keywords=ruby]']
    => Returns all book entries whose keywords attribute equals 'ruby'

### String Replacement

If your bibliography contains BibTeX @string objects, you can let BibTeX-Ruby
replace the strings for you. You have access to a bibliography's strings via
**BibTeX::Bibliography#strings** and you can replace the strings of an entry using
the **BibTeX::Entry#replace!** method. Thus, to replace all strings defined in your
bibliography object **bib** your could use this code:

    bib.entries.each do |entry|
      entry.replace!(bib.strings)
    end
    
A shorthand version for replacing all strings in a given bibliography is the
`Bibliography#replace_strings` method. Similarly, you can use the
`Bibliography#join_strings` method to join individual strings together. For instance:

    > bib = BibTeX::Bibliography.new
    > bib.add BibTeX::Element.parse '@string{ foo = "foo" }'
    > bib.add BibTeX::Element.parse '@string{ bar = "bar" }'
    > bib.add BibTeX::Element.parse <<-END
    >  @book{abook,
    >    author = foo # "Author",
    >    title = foo # bar
    >  }
    > END
    > puts bib[:abook].to_s
    @book{abook,
      author = foo # "Author",
      title = foo # bar
    }
    > bib.replace_strings
    > puts bib[:abook].to_s
    @book{abook,
      author = "foo" # "Author",
      title = "foo" # "bar"
    }
    > bib.join_strings
    @book{abook,
      author = {fooAuthor},
      title = {foobar}
    }

### Conversions

Furthermore, BibTeX-Ruby allows you to export your bibliography for processing
by other tools. Currently supported formats include YAML, JSON, and XML.
Of course, you can also export your bibliography back to BibTeX; if you include
`:meta_comments', your export should be identical to the original '.bib' file,
except for whitespace, blank lines and letter case (BibTeX-Ruby will downcase
all keys).

In order to export your bibliography use **#to\_s**, **#to\_yaml**, **#to\_json**, or
**#to\_xml**, respectively. For example, the following line constitutes a simple
BibTeX to YAML converter:

    BibTeX.open('example.bib').to_yaml

Look at the 'examples' directory for more elaborate examples of a BibTeX to YAML
and a BibTeX to HTML converter.


The Parser
----------

The BibTeX-Ruby parser is generated using the awesome
[racc](http://i.loveruby.net/en/projects/racc/) parser generator. You can take
look at the grammar definition in the file
[lib/bibtex/bibtex.y](https://github.com/inukshuk/bibtex-ruby/blob/master/lib/bibtex/bibtex.y).

For more information about the BibTeX format and the parser's idiosyncrasies
[refer to the project wiki](https://github.com/inukshuk/bibtex-ruby/wiki/The-BibTeX-Format).


Credits
-------

The BibTeX-Ruby package was written by [Sylvester Keil](http://sylvester.keil.or.at/);
kudos and thanks to all [contributors](https://github.com/inukshuk/bibtex-ruby/contributors)!