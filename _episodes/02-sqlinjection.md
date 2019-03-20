---
title: "SQL Injection Attack"
teaching: 10
exercises: 30
questions:
- "What is SQL Injection?"
- "What causes this vulnerability?"
- "What are the broader impacts of SQL Injection attacks?"
- "How can code be secured against SQL Injection attacks?"
objectives:
- "Describe SQL Injection attacks, their possible causes and fixes"
- "Follow instructions to exploit an unsecure database client"
- "Follow instructions to fix the unsecure database client, and verify the fix"
keypoints:
- "SQL Injection attacks exploit database code that does not perform input validation!"
---

## SQL Injection Attacks

The SQL (Structured Query Language) Injection bug exploits the vulnerability in database
client code that does not properly validate user inputs. SQL is a computer language for
performing query, insertion, update, and deletion operations on the data stored in a database.
The SQL syntax also allows for several commands to be chained together, or to ignore the rest
of a command using a comment character. Database client code that simply substitutes in a
user’s input without validation can be exploited by a malicious user to run arbitrary SQL commands.
Since databases often store privileged customer data in many applications, or implement access
control for applications, a malicious user can either retrieve the privileged data, or gain
access to secured resources.

### Demonstration

The demonstration on CHEESEHub illustrates SQL Injection attacks using a single machine that 
contains a barebones database containing a table of user information and an unsecure database 
client application. The client application validates a *username, password* pair against the 
database table and returns the details of the user if successful. The demonstration will illustrate 
how various SQL commands can be executed that either circumvent this validation, insert new records, 
or delete records from the table. Finally, the client code will be modified to add simple input 
validation, which will be tested to ensure that the database is protected against such SQL Injection 
attacks. This demonstration is not intended as a complete account of all possible SQL injection attacks,
neither does it provide the solution to all such attacks. It primarily demonstrates one possible
vulnerability in database client code that developers need to be aware of, and, a specific solution
that works for the client code in question.

> ## Getting Started
> 
> You will need to create an account on [CHEESEHub](https://www.hub.cheesehub.org) to work through this exercise.
> The container in this demonstration has a web interface and is accessible through your web browser, no other special software 
> is needed.
{: .callout} 

We will start by first adding the SQL Injection application:

![Add SQLInjection]({{ page.root }}/fig/sqlinjection/add-application.png)

Next, click the *View* link to go to the application-specific page and start the application container:

![Start Container]({{ page.root }}/fig/sqlinjection/start-container.png)

> ## Container Launch Errors
>
> If the container fails to start, you will see a flashing warning icon next to the container's name. Take a look at the logs to 
> determine the cause of failure and try deleting the application and restarting.
{: .callout}

Once the container has started, launch the container's web interface in a separate browser tab by clicking the icon 
next to the container's name:

![Launch Container]({{ page.root }}/fig/sqlinjection/launch-container.png)

On this new browser tab, click on **SQLInjection.ipynb** to open a Jupyter notebook: 

![Open SQLInjection Notebook]({{ page.root }}/fig/sqlinjection/start-notebook.png)

The notebook contains instructions on conducting various SQL Injection attacks as well as securing the database code to prevent such attacks:

![Follow Instructions]({{ page.root }}/fig/sqlinjection/follow-instructions.png)

The first step is to launch a terminal window to start the database client:

![Launch Terminal]({{ page.root }}/fig/sqlinjection/launch-terminal.png)

At the terminal prompt, start *EmailCloud*, the database client application:

~~~
jovyan@sqlinjection:~$ EmailCloud
~~~
{: .language-bash}

~~~
===============================================================================
		******	*	*	*	***	*
		*	**     **      * *	 *	*		
		******  * *   * *     *****      *      *
		*	* *   * *     *   *      *      *
		******  *   *   *     *   *     ***     *****

		******  *   	   *****     *	  *     ****
		*	*	   *   *     *	  *	*   *
		*	*	   *   *     *	  *	*   *
		*	*	   *   *     *	  *	*   *
		******  *****	   *****     ******     ****

Welcome to Email Cloud, you can store your email addresses in our service.
You can log into our server and get all your email addresses at any time.
===============================================================================


Please log in:
Username:

~~~
{: .output}

Follow the instructions to first try a valid username/password combination:

~~~
...
Please log in:
Username: Alice
Password:
Your profile data is listed in the following:
 [('Alice', 'pwd', 'alice@mail.com'), ('Alice', 'pwd', 'second@mail.com'), ('Alice', 'pwd', 'third@mail.com')]

Try again:
Username:
~~~
{: .output}

Next, try a malicious input that always evaluates to true for each record in the database table:

~~~
Try again:
Username: a' OR '1=1
Password:
Your profile data is listed in the following:
 [('ALice', 'pwd', 'alice@mail.com'), ('Alice', 'pwd', 'second@mail.com'), ('Alice', 'pwd', 'third@mail.com'), ('Bob', 'pwd', 'bob1@mail.com'), ('Bob', 'pwd', 'bob2@mail.com'), ('Charles', 'pwd', 'charlie@mail.com'), ('Donald', 'pwd', 'foo@mail.com'), ('Donald', 'pwd', 'bar@mail.com'), ('Apple', 'pwd', 'johnny@mail.com'), ('Eric', 'pwd', 'xing@mail.com'), ('Eric', 'pwd', 'mello@mail.com')]

Try again:
Username:
~~~
{: .output}

> ## Malicious Inputs
> 
> When trying the *malicious* inputs, pay attention to the special characters being used (single quotes, semicolons, etc.). These special characters exploit knowledge of SQL syntax to get around the naive username, password validation in the query.
{: .callout} 

You can always exit the program by typing *ctrl-c* at the *Username:* prompt:

~~~
Try again:
Username: ^C
exit

Try again:
jovyan@sqlinjection:~$
~~~
{: .output}

We will now modify the database client code to secure it against the vulnerability. Go back to the Jupyter file listing tab in your browser and navigate to the *app* folder, then click on *client.py* to open it:

![Go to App]({{ page.root }}/fig/sqlinjection/goto-client-code.png)

![Go to Client]({{ page.root }}/fig/sqlinjection/goto-client-code2.png)

Look for the lines marked *unsafe* in the *client.py* file; the *safe* alternative is right below these lines:

~~~
	# unsafe
	command = "SELECT * FROM emailcloud WHERE username = '" + username + "' AND password = '" + password "'"
	sqlite.execute(c, command)

	# SQL injection protection	
	# command = "SELECT * FROM emailcloud WHERE username = ? AND password = ?"
	# c.execute(command, (username, password))
~~~
{: .language-python}

Comment out the *unsafe* lines and uncomment the *safe* lines as below:

~~~
	# unsafe
	# command = "SELECT * FROM emailcloud WHERE username = '" + username + "' AND password = '" + password "'"
	# sqlite.execute(c, command)

	# SQL injection protection	
	command = "SELECT * FROM emailcloud WHERE username = ? AND password = ?"
	c.execute(command, (username, password))
~~~
{: .language-python}

Save the changes to the *client.py* file:

![Save Client]({{ page.root }}/fig/sqlinjection/save-file.png)

Now, go back to the terminal and re-run the *EmailCloud* application. Try a malicious input from before:

~~~
Username: a' OR '1=1
Password:
Invalid username password pair!
~~~
{: .output}

> ## How did our change to the client secure it?
> By using parameter substitution for the username and password, instead of directly modifying the query string, we prevent SQL commands from being run as part of a query operation.
{: .callout}

{% include links.md %}
