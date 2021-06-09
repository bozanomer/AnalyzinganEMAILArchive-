Analyzing an EMAIL Archive from gmane and vizualizing the data
using the D3 JavaScript library

This is a set of tools that allow you to pull down an archive
of a gmane repository using the instructions at:

http://gmane.org/export.php

In order not to overwhelm the gmane.org server, The Dr. Chuck has put up
his own copy of the messages at:

http://mbox.dr-chuck.net/

This server will be faster and take a lot of load off the
gmane.org server.

You should install the SQLite browser to view and modify the databases from:

http://sqlitebrowser.org/

The first step is to spider the gmane repository.  The base URL
is hard-coded in the gmane.py and is hard-coded to the Sakai
developer list.  You can spider another repository by changing that
base url.   Make sure to delete the content.sqlite file if you
switch the base url.  The gmane.py file operates as a spider in
that it runs slowly and retrieves one mail message per second so
as to avoid getting throttled by gmane.org.   It stores all of
its data in a database and can be interrupted and re-started
as often as needed.   It may take many hours to pull all the data
down.  So you may need to restart several times.

To give you a head-start, I have put up 600MB of pre-spidered Sakai
email here:

https://www.py4e.com/data_space/content.sqlite.zip

If you download this, you can "catch up with the latest" by
running gmane.py.

Navigate to the folder where you extracted the gmane.zip

Note: Windows has difficulty in displaying UTF-8 characters
in the console so for each console window you open, you may need
to type the following command before running this code:

    chcp 65001

http://stackoverflow.com/questions/388490/unicode-characters-in-windows-command-line-how

Here is a run of gmane.py getting the last five messages of the
sakai developer list:

Mac: python3 gmane.py
Win: gmane.py

How many messages:10
http://mbox.dr-chuck.net/sakai.devel/59643/59644 17553
    matthew@longsight.com 2015-03-20T16:27:12-04:00 re: [building sakai] sakai 10 bulding error
http://mbox.dr-chuck.net/sakai.devel/59644/59645 13128
    alberto.olivamolina@gmail.com 2015-03-20T16:36:12+01:00 re: [building sakai] sakai 10 bulding error
http://mbox.dr-chuck.net/sakai.devel/59645/59646 7557
    eric.duquenoy@univ-littoral.fr 2015-03-20T16:52:24+01:00 [building sakai] lti and sakai groups (or sections)
http://mbox.dr-chuck.net/sakai.devel/59646/59647 1


Did not find From
http://mbox.dr-chuck.net/sakai.devel/59647/59648 9467
    csev@umich.edu 2015-03-20T23:04:53-04:00 re: [building sakai] lti and sakai groups (or sections)
http://mbox.dr-chuck.net/sakai.devel/59648/59649 11727
    matthew@longsight.com 2015-03-20T23:12:47-04:00 re: [building sakai] lti and sakai groups (or sections)
http://mbox.dr-chuck.net/sakai.devel/59649/59650 5916
    blshrestha@mum.edu 2015-03-20T23:42:44-05:00 [using sakai] bulk picture import
http://mbox.dr-chuck.net/sakai.devel/59650/59651 1


Did not find From
http://mbox.dr-chuck.net/sakai.devel/59651/59652 13529
    steve.swinsburg@gmail.com 2015-03-21T07:22:35+11:00 re: [building sakai] sakai 10 bulding error
http://mbox.dr-chuck.net/sakai.devel/59652/59653 25324
    alberto.olivamolina@gmail.com 2015-03-21T11:14:44+01:00 re: [building sakai] sakai 10 bulding error

The program scans content.sqlite from 1 up to the first message number not
already spidered and starts spidering at that message.  It continues spidering
until it has spidered the desired number of messages or it reaches a page
that does not appear to be a properly formatted message.

Sometimes gmane.org is missing a message.  Perhaps administrators can delete messages
or perhaps they get lost.   If your spider stops, and it seems it has hit
a missing message, go into the SQLite Manager and add a row with the missing id - leave
all the other fields blank - and then restart gmane.py.   This will unstick the
spidering process and allow it to continue.  These empty messages will be ignored in the next
phase of the process.

One nice thing is that once you have spidered all of the messages and have them in
content.sqlite, you can run gmane.py again to get new messages as they get sent to the
list.  gmane.py will quickly scan to the end of the already-spidered pages and check
if there are new messages and then quickly retrieve those messages and add them
to content.sqlite.

The content.sqlite data is pretty raw, with an innefficient data model, and not compressed.
This is intentional as it allows you to look at content.sqlite to debug the process.
It would be a bad idea to run any queries against this database as they would be
slow.

The second process is running the program gmodel.py.  gmodel.py reads the rough/raw
data from content.sqlite and produces a cleaned-up and well-modeled version of the
data in the file index.sqlite.  The file index.sqlite will be much smaller (often 10X
smaller) than content.sqlite because it also compresses the header and body text.

Each time gmodel.py runs - it completely wipes out and re-builds index.sqlite, allowing
you to adjust its parameters and edit the mapping tables in content.sqlite to tweak the
data cleaning process.

Running gmodel.py works as follows:

Mac: python3 gmodel.py
Win: gmodel.py

Loaded allsenders 1771 and mapping 29 dns mapping 1
1 2005-12-08T23:34:30-06:00 ggolden22@mac.com
251 2005-12-22T10:03:20-08:00 tpamsler@ucdavis.edu
501 2006-01-12T11:17:34-05:00 lance@indiana.edu
751 2006-01-24T11:13:28-08:00 vrajgopalan@ucmerced.edu
1001 2006-02-02T08:27:30-07:00 john.ellis@rsmart.com
1251 2006-02-15T19:13:17-05:00 ggolden22@mac.com
1501 2006-02-23T16:49:00-05:00 csev@umich.edu
...

The gmodel.py program does a number of data cleaing steps

Domain names are truncated to two levels for .com, .org, .edu, and .net
other domain names are truncated to three levels.  So si.umich.edu becomes
umich.edu and caret.cam.ac.uk becomes cam.ac.uk.   Also mail addresses are
forced to lower case and some of the @gmane.org address like the following

   arwhyte-63aXycvo3TyHXe+LvDLADg@public.gmane.org

are converted to the real address whenever there is a matching real email
address elsewhere in the message corpus.

If you look in the content.sqlite database there are two tables that allow
you to map both domain names and individual email addresses that change over
the lifetime of the email list.  For example, Steve Githens used the following
email addresses over the life of the Sakai developer list:

s-githens@northwestern.edu
sgithens@cam.ac.uk
swgithen@mtu.edu

We can add two entries to the Mapping table

s-githens@northwestern.edu ->  swgithen@mtu.edu
sgithens@cam.ac.uk -> swgithen@mtu.edu

And so all the mail messages will be collected under one sender even if
they used several email addresses over the lifetime of the mailing list.

You can also make similar entries in the DNSMapping table if there are multiple
DNS names you want mapped to a single DNS.  In the Sakai data I add the following
mapping:

iupui.edu -> indiana.edu

So all the folks from the various Indiana University campuses are tracked together

You can re-run the gmodel.py over and over as you look at the data, and add mappings
to make the data cleaner and cleaner.   When you are done, you will have a nicely
indexed version of the email in index.sqlite.   This is the file to use to do data
analysis.   With this file, data analysis will be really quick.

The first, simplest data analysis is to do a "who does the most" and "which
organzation does the most"?  This is done using gbasic.py:

Mac: python3 gbasic.py
Win: gbasic.py

How many to dump? 5
Loaded messages= 58957 subjects= 29059 senders= 1765

Top 5 Email list participants
steve.swinsburg@swinsborg.com 3301
azeckoski@unicon.net 1907
ian@cam.ac.uk 1591
csev@umich.edu 1468
david.horwitz@uct.ac.za 1221

Top 5 Email list organizations
umich.edu 6782
gmail.com 5585
swinsborg.com 3301
cam.ac.uk 2626
uct.ac.za 2576

You can look at the data in index.sqlite and if you find a problem, you
can update the Mapping table and DNSMapping table in content.sqlite and
re-run gmodel.py.

There is a simple vizualization of the word frequence in the subject lines
in the file gword.py:

Mac: python3 gword.py
Win: gword.py

Range of counts: 43177 324
Output written to gword.js

This produces the file gword.js which you can visualize using the file
gword.htm.

A second visualization is in gline.py.  It visualizes email participation by
organizations over time.

Mac: python3 gline.py
Win: gline.py

Loaded messages= 58957 senders= 1765
Top 10 Organizations
['umich.edu', 'gmail.com', 'swinsborg.com', 'cam.ac.uk', 'uct.ac.za', 'indiana.edu', 'unicon.net', 'longsight.com', 'ox.ac.uk', 'berkeley.edu']
Output written to gline.js

Its output is written to gline.js which is visualized using gline.htm
