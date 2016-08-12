# LinuxCon: Visualize Your Code Repos and More with Gource

**NOTE: This is work in progress as I prepare for my talks.
Right now, this repo is a mess as I pull things together from other talks
and add new material.**

By: [Dawn M. Foster](http://fastwonderblog.com). 
PhD student at the [University of Greenwich, Centre for Business Network Analysis](http://www2.gre.ac.uk/about/faculty/business/research/centres/cbna/home) 
and consultant at [The Scale Factory](http://www.scalefactory.com/)


### LinuxCon North America - Toronto 2016

* [Session Info and Abstract](http://sched.co/7JWe)
* Presentation Slides - coming soon
* When: Tuesday, August 23. 2:10pm - 3:00pm
* Room: Marine

### LinuxCon Europe - Berlin 2016

* [Session Info and Abstract](http://sched.co/7o95)
* Presentation Slides - coming soon
* When: Thursday, October 6. 11:15am - 12:05pm
* Room: Potsdam III

Additional Info
---------------

My slides will be available on [slideshare](http://www.slideshare.net/geekygirldawn)
shortly after the presentations.

This github repo contains all of the scripts, data, instructions, and other 
technical materials that you can use to reproduce what I did in the sessions.
If I do this well, you'll be able to use these tools to gather data and 
perform similar analysis using your own community data. 

This README file is where you can find most of the instructions for replicating the
data and analysis from the presentations along with some extra material.

Using Gource
-------

More info about [Gource](https://github.com/acaudwell/Gource), 
including downloads and information about installation. You can 
also look at the presentation I did about Gource at the
[FLOSS Community Metrics Meeting](http://www.slideshare.net/geekygirldawn/floss-community-metrics-gource-custom-log-formats)

Code Repositories
-----------------

If you've never run Gource on your code repositories, you should!

    $ gource </path/to/repo> 

I recommend playing around with the different controls to speed things up / slow down or show / hide
things to get something that looks good with your data. Most of this information can be found
using gource -H, but the [control page](https://github.com/acaudwell/Gource/wiki/Controls)
on the wiki has more details about the controls. You might also check out these [templates]
(https://github.com/FOSSRIT/gourciferous/tree/develop/Templates) with recommended configurations
for different types of data (large projects, long-lived projects, etc.)

You can also check out [Gourciferous](https://github.com/FOSSRIT/gourciferous) for visualizing multiple
repos in a single visualization using the custom log format.


Data Gathering to use the Gource Custom Log Format
--------------------------------------------------

More details about Using the Gource [custom log format](https://code.google.com/p/gource/wiki/CustomLogFormat)
option.

If you are gathing data from open source communities, your best friend is
the [MetricsGrimoire](https://github.com/MetricsGrimoire) suite of tools,
especially [mlstats](https://github.com/MetricsGrimoire/MailingListStats) for mailing lists
and [CVSAnaly](https://github.com/MetricsGrimoire/CVSAnalY) for code repos.

Mailing Lists
-------------

**Step 1: Get your mailing list data into a database using mlstats.**

a) Install [mlstats](https://github.com/MetricsGrimoire/MailingListStats)

    $ python setup.py install

b) Log into mysql and create the database

    mysql> create database mlstats;

c) Import your mailing list data by running mlstats

    $ mlstats --db-user=USERNAME --db-password=PASS http://URLOFYOURLIST

Note: this can take a long time depending on how long your mailing list 
has been around and the number of messages.

**Step 2: Run database queries to extract your data**

This is the "do it yourself" method and requires a bit manual / scripting work on your part. See Step 2 (alternative) below for a Python script that does this for you.

A good list of starter queries can be found on the 
[mlstats wiki](https://github.com/MetricsGrimoire/MailingListStats/wiki/Queries) and
you'll want to look at the [database schema](https://github.com/MetricsGrimoire/MailingListStats/wiki/Database-Schema) as well

To get the data for the Gource custom log, you would need something more like this,
but you would need to re-format it into a pipe-separated file that Gource can read 
(see Python script alternative below):

    SELECT unix_timestamp(DATE_ADD(m.first_date, interval m.first_date_tz second)) 
    AS unix_date, mp.email_address AS sender, (SELECT mp2.email_address FROM 
    messages m2, messages_people mp2 WHERE m2.is_response_of=m.is_response_of AND 
    mp2.message_id=m2.is_response_of limit 1) AS receiver FROM messages_people mp, 
    messages m WHERE YEAR(m.first_date)=2015 AND MONTH(m.first_date)=1 AND 
    mp.message_id=m.message_id; 

**Step 2 (alternative): Use a Python script to easily run the database query and re-format the data a bit.**

**NOTE: STRIP OUT NETWORK PORTION OF THIS SCRIPT**

Run linuxcon.py:

    $ linuxcon.py -o <outputfiledir> -d <database> -u <user-mysql> -p <password-mysql>

What linuxcon.py does:

* Gathers information about where to put output files and the database being used.
* Runs the query.
* Strips the email down to the username (everything before @example.com) to have a 
shorter identifier for the users (looks much better on graphs).
* Formats the output into a nice, clean CSV file called network_output.csv excluding
new threads and self-replies.
* Formats the output into a nice Gource custom log format sorted by time
as gource_output.log

CAVEAT: I am not a real programmer; the code is ugly; and it may or may not work for 
you without some tweaking. However, I will be doing more Python programming, and I 
appreciate **helpful** suggestions about how to improve :)

**Gource - Mailing List Options**

I also added a few other options to make it look a bit nicer:

* -i : Time files remain idle (default 0). This allows people being replied to 
       to disappear after 10 seconds to clean up a bit and make it more readable.
* --max-user-speed : Speed users can travel per second (default: 500). I slowed 
       this down to 100 to make it easier to see the users.
* -a 1 : Auto skip to next entry if nothing happens for a number of seconds (default: 3)
         sped this up a bit.
* --highlight-users : keeps the usernames for the people sending emails from
                      disappearing. I would have liked to have the same for filenames
                      which are the people being replied to, but can't seem to find
                      at option for that
* -s 30 : seconds per day. Not used here, but when you run these yourself, you might
          want to slow things down a bit so you can better see what's going on.

    $ gource -i 5 --max-user-speed 100 -a 1 --highlight-users gource_output.log

Bug Data
--------

ADD DETAILS HERE

Data and Examples
-----------------

**Mailing List Data**

The data used in the examples comes from the Linux kernel 
[IOMMU list](http://lists.linuxfoundation.org/pipermail/iommu/) for January 2015.
[IOMMU](https://en.wikipedia.org/wiki/IOMMU) (Input/Output Memory Management Unit) is used in 
the Linux kernel to map the virtual memory accessible by devices to physical memory for security
and other reasons. I selected this list for a few reasons:

* I'm studying the Linux kernel community at the University of Greenwich, so I wanted to pick something 
from the Linux kernel, but there are [over 150 mailing lists](http://vger.kernel.org/vger-lists.html)
documented for the Linux kernel (and not all of them are listed at that link).  
* The IOMMU list is lower volume than many of the kernel lists (651 posts for January), so it's more
manageable as an example.
* It is a mailman list, which has nicely formatted, clean logs that import well using mlstats.
Many of the kernel lists have other ways of logging (Gmane, etc.) that aren't quite as clean
when imported. 

**Bug Data**

The bug data is courtesy of [Dr. Guido Conaldi](http://www.gre.ac.uk/business/study/ibe/staff/guido-conaldi), Senior Lecturer at The University of Greenwich. This dataset came from one of his research projects using network analysis on bug tracker data.

ADD DETAILS HERE.

License and Copyright
---------------------

Code is licensed under [GNU General Public License (GPL), version 3 or later](http://www.gnu.org/licenses/gpl.txt).

Other content, including the tutorial materials in this README and data files are licensed under a 
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/).

Copyright (C) 2015 Dawn M. Foster
