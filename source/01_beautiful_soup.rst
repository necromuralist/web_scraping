Beautiful Soup Introduction
===========================

.. contents::


1 Introduction
--------------

This is a basic introduction to using `Beautiful Soup <https://www.crummy.com/software/BeautifulSoup/>`_ to extract text from web pages.

2 Imports and Set Up
--------------------

.. code:: ipython

    # pypi
    import requests
    from bs4 import BeautifulSoup

We're going to be pulling the pages from the website set up for *Web Scraping With Python*.

.. code:: ipython

    URL = "http://www.pythonscraping.com/pages/"

3 Loading the Page
------------------

This loads ``page1`` of the ``www.pythonscraping.com`` into a :py:class:`bs4.BeautifulSoup` object and prints the first headline.

.. code:: ipython

    response = requests.get(URL + "page1.html")
    soup = BeautifulSoup(response.text)
    print(soup.h1)

::

    <h1>An Interesting Title</h1>

4 Using CSS
-----------

Because CSS tags classify what a section of HTML is, you can sometimes use it to sort out tags. This next example looks at a page that has the beginning of *War and Peace* with the dialog colored red and the names of characters colored green using css span tags.

\.. ode:: html

'<span class="green">Anna Pavlovna</span>'

First we can grab the page and load it into Beautiful Soup.

.. code:: ipython

    response = requests.get(URL + "warandpeace.html")
    assert response.ok
    soup = BeautifulSoup(response.text)

To get all the character names we can use beautiful soup to find all the text surrounded by a ``span`` tag with the "green" class. I'll just show the top-five.

.. code:: ipython

    names = soup.findAll("span", {"class": "green"})
    for name in names[:5]:
        print(name.get_text().replace("\n", " "))

::

    Anna Pavlovna Scherer
    Empress Marya Fedorovna
    Prince Vasili Kuragin
    Anna Pavlovna
    St. Petersburg

So, the first thing we can see is that I described the content of the tags incorrectly, since *St. Petersburg* is a place, not a character name (I think, I haven't actually read the text). The other thing to notice is that some of the names had newline characters in them (which is why I'm stripping them out before printing), so even after grabbing the text, there's still some post processing to do.

Another thing to notice is that calling ``BeautifulSoup.get_text`` only returns the text within the tags, the tags themselves are stripped out. It will do this even if you grab a block of HTML that has nested HMTL tags within it.
