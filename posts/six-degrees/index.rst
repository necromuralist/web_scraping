.. title: Six Degrees
.. slug: six-degrees
.. date: 2017-11-10 12:36:10 UTC-08:00
.. tags: webscraping
.. category: experiment, how-to
.. link: 
.. description: Six Degrees of Wikipedia
.. type: text

1 Introduction
--------------

This is a basic introduction to making a web-crawler. It will start from a given `wikipedia <https://en.wikipedia.org/wiki/Main_Page>`_ page and randomly pick pages to go to (up to six).

2 Imports
---------

.. code:: ipython

    # python standard library
    from urllib.parse import urljoin
    import random
    import re

    # pypi
    from bs4 import BeautifulSoup
    import requests

3 The URLs
----------

The format for wikipedia URLs is the base URL ("`https://en.wikipedia.org/wiki/ <https://en.wikipedia.org/wiki/>`_") followed by the subject. If there are multiple words in the subject they are separated by underscores, e.g. "`https://en.wikipedia.org/wiki/Ophiocordyceps_unilateralis <https://en.wikipedia.org/wiki/Ophiocordyceps_unilateralis>`_". It will automatically set the first letter of the first word of the subject to a capital letter, but not the second word. So if it is a proper name it's case sensitive, but in some cases it isn't. 

There are also special urls that start with a word followed by a colon (like "Category:") that we probably want to exclude.

.. code:: ipython

    WIKI_URL = "https://en.wikipedia.org"
    BASE_PAGE = "/wiki/Main_Page"
    LINK = re.compile("^/wiki/\w+[^:]\w*$")
    OUTPUT = "{}\t[[{}][{}]]"

4 A getter
----------

This is a function to get the page. It checks for errors and raises an error if there was one.

.. code:: ipython

    def get_page(page):
        """Gets the page
    
        Args:
         page (str): name of page (not the full URL)
    
        Returns:
         BeautifulSoup: the loaded page

        Raises:
         RuntimeError: the page wasn't successfully loaded
        """
        url = urljoin(WIKI_URL, page)
        response = requests.get(url)
        if not response.ok:
            raise RuntimeError("Unable to get {}".format(url))
        return BeautifulSoup(response.text, "html5lib")

5 Next Page
-----------

This will pick a random page from the one given.

.. code:: ipython

    def next_page(article):
        """Gets a random page

        Args:
         article (str): article link of the form '/wiki/<article>'

        Returns:
         str: random article link from the page
        """
        soup = get_page(article)
        links = (soup.find("div", {"id": "bodyContent"})
                 .findAll("a", href=LINK))
        return links[random.randrange(len(links))]

6 Grabbing the pages
--------------------

.. code:: ipython

    def grab_pages(start=BASE_PAGE, count=6):
        """grabs random pages and lists their titles

        Args:
         start (str): article to start at
         count(int): number of pages to grab

        Returns:
         :class:`bs4.element.Tag`: last page found
        """
        page = next_page(start)
        print(OUTPUT.format(1, urljoin(WIKI_URL, page["href"]), page.attrs["title"]))
        for number in range(2, count+1):
            page = next_page(page.attrs["href"])
            print(OUTPUT.format(number, urljoin(WIKI_URL, page["href"]), page.attrs["title"]))
        return page

.. code:: ipython

    def to_path(article):
        """converts the article to /wiki/article

        Args:
         article (str): article to find

        Returns:
         str: path to article
        """
        return urljoin("/wiki/", article)

6.1 First, a random six pages starting at the main Wikipedia page
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: ipython

    grab_pages()

1	`Tropical wave <https://en.wikipedia.org/wiki/Tropical_wave>`_
2	`Tropics <https://en.wikipedia.org/wiki/Tropics>`_
3	`Bora Bora <https://en.wikipedia.org/wiki/Bora_Bora>`_
4	`Menehune <https://en.wikipedia.org/wiki/Menehune>`_
5	`United Airlines <https://en.wikipedia.org/wiki/United_Airlines>`_
6	`Grand Canyon Airlines <https://en.wikipedia.org/wiki/Grand_Canyon_Airlines>`_

6.2 Now random pages starting with Kevin Bacon's page
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: ipython

    grab_pages(to_path("Kevin Bacon"))

1	`Mark Ruffalo <https://en.wikipedia.org/wiki/Mark_Ruffalo>`_
2	`Frances de la Tour <https://en.wikipedia.org/wiki/Frances_de_la_Tour>`_
3	`Alice Ghostley <https://en.wikipedia.org/wiki/Alice_Ghostley>`_
4	`Kaye Ballard <https://en.wikipedia.org/wiki/Kaye_Ballard>`_
5	`Follies <https://en.wikipedia.org/wiki/Follies>`_
6	`Arthur Rubin <https://en.wikipedia.org/wiki/Arthur_Rubin>`_

6.3 Some random 20 pages from the year 1968
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: ipython

    grab_pages(to_path("1968"), 20)

1	`Bert Wheeler <https://en.wikipedia.org/wiki/Bert_Wheeler>`_
2	`Broadway theatre <https://en.wikipedia.org/wiki/Broadway_theatre>`_
3	`Palmer's Theatre <https://en.wikipedia.org/wiki/Palmer's_Theatre>`_
4	`Fulton Theatre <https://en.wikipedia.org/wiki/Fulton_Theatre>`_
5	`44th Street Theatre <https://en.wikipedia.org/wiki/44th_Street_Theatre>`_
6	`Longacre Theatre <https://en.wikipedia.org/wiki/Longacre_Theatre>`_
7	`St. James Theatre <https://en.wikipedia.org/wiki/St._James_Theatre>`_
8	`Native Son <https://en.wikipedia.org/wiki/Native_Son>`_
9	`Modern Library List of Best 20th-Century Novels <https://en.wikipedia.org/wiki/Modern_Library_List_of_Best_20th-Century_Novels>`_
10	`Scientology <https://en.wikipedia.org/wiki/Scientology>`_
11	`Cult Awareness Network <https://en.wikipedia.org/wiki/Cult_Awareness_Network>`_
12	`Fight Against Coercive Tactics Network <https://en.wikipedia.org/wiki/Fight_Against_Coercive_Tactics_Network>`_
13	`Fishman Affidavit <https://en.wikipedia.org/wiki/Fishman_Affidavit>`_
14	`Church of Scientology of California v. Armstrong <https://en.wikipedia.org/wiki/Church_of_Scientology_of_California_v._Armstrong>`_
15	`Celebrity Centres <https://en.wikipedia.org/wiki/Celebrity_Centres>`_
16	`United States v. Hubbard <https://en.wikipedia.org/wiki/United_States_v._Hubbard>`_
17	`Scientology Missions International <https://en.wikipedia.org/wiki/Scientology_Missions_International>`_
18	`Clearwater Hearings <https://en.wikipedia.org/wiki/Clearwater_Hearings>`_
19	`Keeping Scientology Working <https://en.wikipedia.org/wiki/Keeping_Scientology_Working>`_
20	`Xenu <https://en.wikipedia.org/wiki/Xenu>`_

**Bert Wheeler** was an American comedian who died in 1968. **Xenu** comes from scientology and is the dictator of the "Galactic Confederacy" who brought some of his people to Earth and killed them - their immortal spirits (thetans) adhere to humans, causing spiritual harm. The biggest jump seems to come from the **Modern Library List of Best 20th Century Novels** to **Scientology**. It looks like it happened because there was a Reader's List that included **Battlefield Earth** by **Scientology** founder L. Ron Hubbard.

6.4 Twenty pages from the zombie fungus
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: ipython

    grab_pages(to_path("Ophiocordyceps_unilateralis"), 20)

1	`PubMed Central <https://en.wikipedia.org/wiki/PubMed_Central>`_
2	`Document Type Definition <https://en.wikipedia.org/wiki/Document_Type_Definition>`_
3	`Parsing <https://en.wikipedia.org/wiki/Parsing>`_
4	`Compiler frontend <https://en.wikipedia.org/wiki/Compiler_frontend>`_
5	`Formal methods <https://en.wikipedia.org/wiki/Formal_methods>`_
6	`Concurrent computing <https://en.wikipedia.org/wiki/Concurrent_computing>`_
7	`Digital object identifier <https://en.wikipedia.org/wiki/Digital_object_identifier>`_
8	`ISO 13399 <https://en.wikipedia.org/wiki/ISO_13399>`_
9	`Antimagnetic watch <https://en.wikipedia.org/wiki/Antimagnetic_watch>`_
10	`MPEG-4 Part 2 <https://en.wikipedia.org/wiki/MPEG-4_Part_2>`_
11	`H.263 <https://en.wikipedia.org/wiki/H.263>`_
12	`MOD and TOD <https://en.wikipedia.org/wiki/MOD_and_TOD>`_
13	`8 mm video format <https://en.wikipedia.org/wiki/8_mm_video_format>`_
14	`HDCAM <https://en.wikipedia.org/wiki/HDCAM#HDCAM_SR>`_
15	`Wayback Machine <https://en.wikipedia.org/wiki/Wayback_Machine>`_
16	`Netnews <https://en.wikipedia.org/wiki/Netnews>`_
17	`Pingback <https://en.wikipedia.org/wiki/Pingback>`_
18	`History of podcasting <https://en.wikipedia.org/wiki/History_of_podcasting>`_
19	`Shiira <https://en.wikipedia.org/wiki/Shiira>`_
20	`Software bug <https://en.wikipedia.org/wiki/Software_bug>`_

This one jumped pretty quickly to computer-related subjets and seems to have gotten stuck there (**Shiira** was the name of a web-browser).
