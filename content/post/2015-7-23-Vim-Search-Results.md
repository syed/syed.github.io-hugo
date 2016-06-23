+++
title = "Better Search in Vim"
date = "2015-07-23"
+++

I find that I searching in Vim is broken. For example, when searching for a variable in my code, I often have no context
where the next variable is. Sure, I can use `n` to go to the next match but I dont't get an overall picture of how and 
where that variable is being used in my file. Also, I cannot easily jump from one position to another without cycling 
through all of them. To fix this, I was playing around with the quickfix window to see if I can use that to better 
browse my matches. 

Here is what I have come up with.I think this works well for me as I can now easily switch between different 
mataches and I also have a context of how my match is being used.  


![Vim qfix search](https://www.dropbox.com/s/b879oj9m2khcep8/vim-qfix-search.png?raw=1 "Vim qfix search")

To enable this, add the following line to your `.vimrc`

{% highlight bash %}

map <F5> /<C-r><C-w> <CR> :vim /<C-r><C-w>/g % \| copen <CR>

{% endhighlight %}

Go over a word and press `F5` to open the quickfix window with the matches.
You can switch between the quickfix buffer and the main buffer by using
<Ctrl-w><w>. 





