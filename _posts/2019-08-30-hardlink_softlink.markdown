---
title: "Linux Hardlink and Softlink/Symlink explained."
categories:
  - blog
tags:
  - Linux_Basics
  - where_to_start
  - Tech
---

What is Inode?
it's the `metadata` about a regular file and directory in Linux.

Let's create a file with some contents.
{% highlight bash %}
# vim myfile.txt
{% endhighlight %}


How to find Inode information of a file?

{% highlight bash %}
# stat myfile.txt
  File: myfile.txt
  Size: 2023      	Blocks: 8          IO Block: 4096   regular file
Device: fd03h/64771d	Inode: 15997895    Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/   antop)   Gid: ( 1000/   antop)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2019-08-30 08:29:16.974661276 +0530
Modify: 2019-08-30 08:29:16.974661276 +0530
Change: 2019-08-30 08:29:16.974661276 +0530
Birth: 2019-08-30 08:29:16.974661276 +0530
{% endhighlight %}


In above inode information it is showing `Links : 1`.


Create a Hard Link to `myfile.txt`:

{% highlight bash %}
# ln myfile.txt myfile_hardlink.txt
{% endhighlight %}

Again check the value of `Links`:
{% highlight bash %}
# stat myfile.txt
  File: myfile.txt
  Size: 2023      	Blocks: 8          IO Block: 4096   regular file
Device: fd03h/64771d	Inode: 15997895    Links: 2
Access: (0644/-rw-r--r--)  Uid: ( 1000/   antop)   Gid: ( 1000/   antop)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2019-08-30 08:29:16.974661276 +0530
Modify: 2019-08-30 08:29:16.974661276 +0530
Change: 2019-08-30 08:29:44.868059994 +0530
 Birth: 2019-08-30 08:29:16.974661276 +0530
{% endhighlight %}


Now Value changed: "Links = 2".

Check the inode information of hardlink we have created (`myfile_hardlink.txt`):

{% highlight bash %}
# stat myfile_hardlink.txt
  File: myfile_hardlink.txt
  Size: 2023      	Blocks: 8          IO Block: 4096   regular file
Device: fd03h/64771d	Inode: 15997895    Links: 2
Access: (0644/-rw-r--r--)  Uid: ( 1000/   antop)   Gid: ( 1000/   antop)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2019-08-30 08:29:16.974661276 +0530
Modify: 2019-08-30 08:29:16.974661276 +0530
Change: 2019-08-30 08:29:44.868059994 +0530
 Birth: 2019-08-30 08:29:16.974661276 +0530
{% endhighlight %}



Create Soft Link/Symlink:
{% highlight bash %}
# ln -s myfile.txt myfile_softlink.txt
{% endhighlight %}


Check and compare inode and size of `myfile.txt` and it's links.
{% highlight bash %}
# ls -il                              
total 8
15997895 -rw-r--r--. 2 antop antop 2023 Aug 30 08:29 myfile_hardlink.txt
16025624 lrwxrwxrwx. 1 antop antop   10 Aug 30 08:30 myfile_softlink.txt -> myfile.txt
15997895 -rw-r--r--. 2 antop antop 2023 Aug 30 08:29 myfile.txt                           # Here 3rd column showing `2` means `Links: 2`
{% endhighlight %}

Now we could see that inode of myfile.txt and myfile_hardlink.txt are same.

![hardlink and softlink](/assets/images/hardlink_softlink.jpg)


Let's continue. Create a copy of hardlink we have created before:
{% highlight bash %}
# cp myfile_hardlink.txt myfile_hardlink_copy.txt
{% endhighlight %}

{% highlight bash %}
# ls -il
total 12
16026660 -rw-r--r--. 1 antop antop 2023 Aug 30 08:31 myfile_hardlink_copy.txt
15997895 -rw-r--r--. 2 antop antop 2023 Aug 30 08:29 myfile_hardlink.txt
16025624 lrwxrwxrwx. 1 antop antop   10 Aug 30 08:30 myfile_softlink.txt -> myfile.txt
15997895 -rw-r--r--. 2 antop antop 2023 Aug 30 08:29 myfile.txt
{% endhighlight %}


So copy mean's, both files don't have any relation. It's a COPY.

Create a copy of softlink:
{% highlight bash %}
# cp myfile_softlink.txt myfile_softlink_copy.txt

# ls -il
total 16
16026660 -rw-r--r--. 1 antop antop 2023 Aug 30 08:31 myfile_hardlink_copy.txt
15997895 -rw-r--r--. 2 antop antop 2023 Aug 30 08:29 myfile_hardlink.txt
16023584 -rw-r--r--. 1 antop antop 2023 Aug 30 08:31 myfile_softlink_copy.txt
16025624 lrwxrwxrwx. 1 antop antop   10 Aug 30 08:30 myfile_softlink.txt -> myfile.txt
15997895 -rw-r--r--. 2 antop antop 2023 Aug 30 08:29 myfile.txt

{% endhighlight %}


And check size of both links and their copies.
{% highlight bash %}
# du -sch *

4.0K	myfile_hardlink_copy.txt
4.0K	myfile_hardlink.txt
4.0K	myfile_softlink_copy.txt
0	myfile_softlink.txt
12K	total
{% endhighlight %}


Copy will create a copy of entire contents. It's a COPY.

Upon creating a hardlink, `Links =2` in inode of the myfile.txt
{% highlight bash %}
# stat myfile.txt|grep Links:
Device: fd03h/64771d	Inode: 15997895    Links: 2
{% endhighlight %}

Let's remove hardlink and check the value of "Links":
{% highlight bash %}
# rm myfile_hardlink.txt

# stat myfile.txt|grep Links:
Device: fd03h/64771d	Inode: 15997895    Links: 1
{% endhighlight %}

Links value become 1 again. So the actual content/file info gets deleted/removed only if there are no `HARDLINKS` present.

See the video demonstration from below:

<script id="asciicast-265029" src="https://asciinema.org/a/265029.js" async></script>
