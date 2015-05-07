# Introduction #

Little known fact - POSIX requires that the `cd` command allow you to change directory into `//` in addition to `/`.

This is [mentioned in section E10 here](http://www.faqs.org/faqs/unix-faq/shell/bash/):
```
E10) Why does `cd //' leave $PWD as `//'?

POSIX.2, in its description of `cd', says that *three* or more leading
slashes may be replaced with a single slash when canonicalizing the
current working directory.

This is, I presume, for historical compatibility.  Certain versions of
Unix, and early network file systems, used paths of the form
//hostname/path to access `path' on server `hostname'.
```

# Rationale #
So I was left with a dilemma - either the command logger strips away the leading extra slash when recording the `${PWD`}, or we offload some complexity to the query writer.

My original thinking was that when troubleshooting strange issues on a system, knowing what the system believes the `${PWD`} is can actually be important.  For example, if a script checks the current working directory and compares with a hard-coded value such as `/home/me` - if `${PWD`} reports `//home/me` and the script fails - this indicates a bug in the script.

I chose to keep the actual value of `${PWD`} for this reason.

# Queries #
Query authors need to deal with the fact that if a user is in a double-slash rooted directory and wishes to view history based on the value of `${PWD`}, in most cases they also want to see history for the single-slash rooted equivalent as well.

For example.
```
  me@my_host:/home/me $ ash_query -q CWD  # I want to see my homedir history
  me@my_host:/home/me $ cd //home/me  # Enter the double-slash-rooted homedir
  me@my_host://home/me $ ash_query -q CWD  # I ALSO want to see my homedir history
```

If the CWD query did not have all three tests of the current working directory, the first and second invocations of `ash_query` in the example above would have totally different results.

Also, if you really want to make this distinction, you can still [write your own query](HOWTO_Explore.md) to do just that!