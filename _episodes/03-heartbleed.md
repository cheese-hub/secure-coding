---
title: "Heartbleed Attack"
teaching: 10
exercises: 30
questions:
- "What is the Heartbleed bug?"
- "What caused this vulnerability?"
- "What are the broader impacts of the Heartbleed bug?"
- "How can code be secured against bugs like Heartbleed?"
objectives:
- "Describe the Heartbleed bug, its cause and fix"
- "Follow instructions to exploit the Heartbleed bug to access a secure web page"
keypoints:
- "Heartbleed was caused by a simple missing input validation step!"
---

## Heartbleed Bug

The Heartbleed bug is an example of a cybersecurity attack that exploits a vulnerability
in the OpenSSL library. Briefly, a missing validation step in the OpenSSL library could allow
a hacker to access sensitive information on a server that is using the vulnerable library.
As part of the handshake protocol for establishing a SSL connection between a client and server,
a heartbeat message is sent from the client, which is then relayed back from the server.
The client is also responsible for sending the length of its heartbeat message, which the server
uses to determine the number of bytes from memory that need to be returned. A spurious length value,
which isn't validated by the server can cause it to return bytes adjacent to the memory location
where the client's heartbeat message is stored. If the server has other applications running,
it may return information saved to memory by any one of these applications as a result of the
Heartbleed bug. 

### Demonstration

The demonstration on CHEESEHub illustrates the Heartbleed attack using three containers; a hacker,
victim, and server. The server serves a barebones webpage that provides a simple authentication form.
If the user enters a valid login, they are presented with a success page. The server also uses
cookies so that a user need not re-login to the server in the same browser session. Both the hacker
and victim are VNC containers that provide a complete Linux desktop environment with a built-in terminal
and web browser. The hacker also contains a simple Python script that performs the Heartbleed attack
against the server. The script parses the response from the server to look for session cookies. When the
victim successfully logs in to the server web application, and visits the application again in their browser,
a session cookie is exchanged. If the hacker is able to retrieve this cookie, they can set the same cookie
in their browser and get to the secured page on the server web application. While this is just one
simple demonstration of a vulnerability that could be addressed using additional validation of the
source IP address etc., the point is to illustrate how access to data from a server's memory can
have far-reaching consequences.

> ## Getting Started
> 
> You will need to create an account on [CHEESEHub](https://www.hub.cheesehub.org) to work through this exercise.
> The container in this demonstration has a web interface and is accessible through your web browser, no other special software 
> is needed.
{: .callout} 

We will start by first adding the Heartbleed application:

![Add Heartbleed]({{ page.root }}/fig/heartbleed/add-application.png)

Next, click the *View* link to go to the application-specific page and start the application containers:

![Start Containers]({{ page.root }}/fig/heartbleed/start-containers.png)

> ## Container Launch Errors
>
> If any of the containers fail to start, you will see a flashing warning icon next to the container's name. Take a look at the logs to 
> determine the cause of failure and try deleting the application and restarting.
{: .callout}

Once all the containers have started, launch the hacker and victim's web interfaces in separate browser tabs by clicking the icon 
next to the container's name:

![Launch Containers]({{ page.root }}/fig/heartbleed/launch-containers.png)

On the hacker browser tab, double click the **Instructions.html** icon on the desktop to open the file:

![Open Hacker Instructions]({{ page.root }}/fig/heartbleed/hacker-instructions.png)

![Read Hacker Instructions]({{ page.root }}/fig/heartbleed/hacker-instructions2.png)

Start the *Firefox* browser in the hacker window from the bottom left corner:

![Launch Hacker Firefox]({{ page.root }}/fig/heartbleed/launch-hacker-firefox.png)

The next step is to open the server's webpage in this browser. First, we will need to determine the IP address of the server.

To do that, click on the *console* icon for the server in the container listing page:

![Launch Server Console]({{ page.root }}/fig/heartbleed/server-console.png)

We can now find the IP address of the server at the terminal prompt:

~~~
# ifconfig
~~~
{: .language-bash}

~~~
eth0	Link encap:Ethernet	HWaddr 3a:6d:b6:bc:ac:b1
	inet addr:10.32.0.7	Bcast:10.47.255.255	Mask:255.240.0.0
	inet6 addr: fe80::386d:b6ff:febc:acb1/64 Scope:Link
	...
~~~
{: .output}

Go to this IP address in the Firefox browser on the hacker. You will be presented with a login page.
Since the hacker is unaware of the valid logins for the server, try some random combination like *admin*, *test*.

![Hacker Incorrect Login]({{ page.root }}/fig/heartbleed/incorrect-user.png)

Now go to the victim browser tab and launch Firefox in that VNC window from the bottom left corner. We will again
go to the server's web page in this Firefox browser. The victim is assumed to possess valid login details. Use the
combination: *foo*, *bar*. 

![Victim Correct Login]({{ page.root }}/fig/heartbleed/correct-user.png)

The browser will then redirect to a simple welcome message:

![Victim Correct Login Redirect]({{ page.root }}/fig/heartbleed/correct-user2.png)

We will now exploit the Heartbleed bug to steal the session cookie from the victim's session. First, open a new
tab in the victim's Firefox browser and go to the server's IP again. You should be presented with the welcome
message instead of having to login again:

![Victim New Tab]({{ page.root }}/fig/heartbleed/victim-new-tab.png)

On the hacker tab, start a new terminal from the bottom left corner:

![Hacker Terminal]({{ page.root }}/fig/heartbleed/hacker-terminal.png)

At the terminal prompt, run the *hacker.py* script that exploits the Heartbleed bug to look for session cookies:

~~~
hacker@heartbleedhacker:~$ python hacker.py 10.32.0.7
~~~
{: .language-bash}

~~~
Scanning 10.32.0.7 on port 443
Connecting...
Sending Client Hello...
Waiting for Server Hello...
 ... received message: type = 22, ver = 0302, length = 66

...


Sending heartbeat request...
 ... received message: type = 34, ver = 0302, length = 16384
Received heartbeat response:
('session token found! ', '.eJw9k....kN4')

WARNING: server 10.32.0.7 returned more data than it should - server is vulnerable!
~~~
{: .output}

> ## Script Success
> 
> It is likely that the script may not succeed and return a cookie immediately. You may need to re-run the script a couple of times until a session cookie is found.
> It is also vital that you run the *hacker.py* script immediately after opening a new tab in the victim's browser and navigating to the server's IP. This will ensure
that you capture the session cookie for that interaction.
{: .callout} 

We will now use this session cookie to circumvent the authentication process. Copy the session token string
from the terminal window. Next, on the hacker's Firefox browser, open a new tab and go to the server's IP address.
Then, open the *Storage Inspector* from the top right corner to view the current session cookies:

![Go to Options]({{ page.root }}/fig/heartbleed/storage-insp1.png)

![Go to Web Developer]({{ page.root }}/fig/heartbleed/storage-insp2.png)

![Go to Storage Inspector]({{ page.root }}/fig/heartbleed/storage-insp3.png)

Find the entry for the server's IP address, and click on the entry for the *Value* field to highlight it. Replace
this value with the string copied over from the terminal window and hit *Enter*.

Type in the server's IP address on this tab's address bar to go back to the server. This time, you should be
presented with the welcome screen, just as in the victim's browser. 

> ## What happened?
> By exploiting the Heartbleed bug, the hacker was able to obtain the session cookie exchanged between the victim and
> server and use it in their own browser session. 
{: .callout}

{% include links.md %}
