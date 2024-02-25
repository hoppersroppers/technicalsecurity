# Linux Process Hunting

<iframe allowfullscreen height="360" src="https://www.youtube.com/embed/O2x9hB4_C7M?wmode=opaque" width="640"></iframe>  

Now that we have detected there is a malicious process running on one of
our hosts, let's hop onto the box and start looking through the
processes.

## Detecting Evil

Let's start looking at all the ways we can detect this and find out more
information, without having to use any networking commands.

In this scenario, let's say our network analysts reported that weird
traffic makes them believe the system has been compromised in the last
week.

Running the "history" command and seeing what pops up is a solid first
move once you get on any box. However, because the attacker cleared out
their history, we shouldn't see anything there, which should be an alert
to us itself!

Using the `find` command we can check for files made in the last week
with `* find / -mtime -o -ctime -7`. To get even more specific, like if
we thought we were recently compromised, we can make that 7 a 1. You
should see a new file or two. Check the user of that file with `ls -l`.
If that user seems suspicious, let's check out their most recent
commands run.

`su` over to the new user you discovered and check their history, maybe
the attacker forgot to hide that as well.

Now that we know there is definitely a new user added and what looks
like crontabs, let's start by looking for common persistence mechanisms
from the command line. First, we can check cronjobs using the `crontab`
command, or `crontab -u` to check other users.

We can also look for SETUID or SETGID binaries with
`find / -perm -4000 -user root -type f` and
`find / -perm -6000 -type f`. Modify those commands as you see fit.

Another great tool to use is going through the logs stored in /var/log.
In the real world, many organizations collect all of these logs from
their computers and store them in a centralized location where analysts
can look through them to find evil, or after evil is detected, determine
what else was done. With that said, just because an organization is
collecting does not mean they are detecting, just like with packet
captures.

There are a ton of different logs in there, but the most useful is named
"syslog". You can look through that and find information about what has
happened, especially if you search through it to find where the attacker
began running commands.

Now that we've discovered that there is definitely malware on the box,
in the real world an analyst would log onto the potentially compromised
host and <a
href="https://blog.apnic.net/2019/10/14/how-to-basic-linux-malware-process-forensics-for-incident-responders/"
rel="noopener" target="_blank">begin doing incident response</a>. For us
though, let's wrap this up.

For other commands useful to look for malicious processes check out <a
href="https://www.sandflysecurity.com/wp-content/uploads/2018/11/Linux.Compromise.Detection.Command.Cheatsheet.pdf"
rel="noopener" target="_blank">this cheatsheet</a>

## Linux Rootkit Hunting

To go one step further, if you fear that you might have a rootkit,
installing and running the `rkhunter` or `chrootkit` tools is an
excellent idea. These tools attempt to look for signatures of known
rootkits or discrepancies in the output of commands which might indicate
the presence of something modifying the operating system.... with that
said, many rootkits have specific anti-detection defenses built-in, so
it is a cat and mouse game.

Assignment:

1.  Write up a minimal report on the malicious actions taken by the
    attacker, as well as information about the backdoor. Include
    information from the command outputs to provide detail.

Congratulations on your first successful hunt! If you are looking to
create a nice addition to your professional portfolio, I recommend
combining your last two assignments, making it look pretty, and posting
it somewhere. People will be very interested in it.
