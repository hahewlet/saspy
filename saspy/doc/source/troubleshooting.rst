
.. Copyright SAS Institute

.. currentmodule:: saspy


===============
Troubleshooting
===============

This chapter covers troubleshooting procedures with this module. While we don't expect you to have trouble,
there are some cases where you might not have everything working right. We've tried to provide an easy reference
for diagnosing and fixing those issues here.


***********************************
Connection and configuration issues
***********************************

Although setup and configuration is pretty simple, if you do have something not quite right, it
may be hard to figure out what's wrong. That's when you come to this chapter.

We've added quite a bit of self diagnostics and error messages for many of the likely issues that can
happen trying to start up a connection to SAS. Each access method has its own set of usual suspects. With a
little help and explaination here, you can probably diagnose and correct any issue you might have.

Problems in this category will be when using the saspy.SASsession() method to connect to a SAS session.
The very first thing to look at is your sascfg_personal.py file (based off the examples in sascfg.py in
the installation directory). This is where the configurations definition are. The sample file itself has
documentation and so does :doc:`install`.


Common diagnostics
------------------

Although each access method has its own ways something can go wrong, there are some common diagnostics
you will get and can use to track down the issue.

The first is that if the SASsession() method fails, it will return any erros it can, as well as the
actual command it was trying to run to connect to SAS. That will vary with access method, but in each
case, you can cut-n-paste that command into a shell on the machine where Python is running
and that may provide more diagnostics and error messages then may have been displayed from SASsession().

For instance, here's a very simple case using the STDIO access method on a local linux machine. The
Configuration Definition is nothing but a valid path which should work.

.. code-block:: ipython3

    default  = {'saspath': '/opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_u8'}

When I try to run I get the following:

.. code-block:: ipython3

    Linux-1> python3.5
    >>> import saspy
    >>> sas = saspy.SASsession()

    SAS Connection failed. No connection established. Double check your settings in sascfg_personal.py file.

    Attempted to run program /opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_u8 with the following parameters:
    ['/opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_u8', '-nodms', '-stdio', '-terminal', '-nosyntaxcheck', '-pagesize', 'MAX', '']

    Try running the following command (where SASPy is running) manually to see if you can get more information on what went wrong:
    /opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_u8 -nodms -stdio -terminal -nosyntaxcheck -pagesize MAX

    No SAS process attached. SAS process has terminated unexpectedly.

I can see from that error it didn't work, but it didn't really tell me why or what to do about it. Well,
it did say I should try running that command to see if I could get better diagnostics. What the heck, let's try:


.. code-block:: ipython3

    Linux-1> /opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_u8 -nodms -stdio -terminal -nosyntaxcheck -pagesize MAX

    ERROR: The current date of Tuesday, March 14, 2017 is past the final
    ERROR: expiration date for your SAS system, which is Monday, January 2, 2017.
    ERROR: Please contact your SAS Installation Representative to obtain your
    ERROR: updated SAS Installation Data (SID) file, which includes SETINIT
    ERROR: information.
    To locate the name of your SAS Installation Representative go to
    http://support.sas.com/repfinder and provide your site number 70068118 and
    company name as Linux for x64 All Compatible Non-Planning Products. On the
    SAS REP list provided, locate the REP for operating system LIN X64.
    ERROR: Initialization of setinit information from SASHELP failed.
    NOTE: Unable to initialize the options subsystem.
    ERROR: (SASXKINI): PHASE 3 KERNEL INITIALIZATION FAILED.
    ERROR: Unable to initialize the SAS kernel.

Well go figure. My SAS license has expired.

The same process can be used with other access methods. Now we'll look at what can be misconfigured for the
various connection methods, see what the errors look like, and how you can determine what the problem is.


STDIO
-----

There are only a couple of things that can go wrong here. First, this only works on Unix, not Windows,
so if you're having problems getting to to work from Windows, well there you go.

Second, the only thing you really need to have right is the path to the SAS startup script in your SAS
installation. The 'saspath' value in your configuration definition needs to be correct and accessible.

If, for instance, you just installed SASPy on your PC and then tried to use it, without configuring it,
you will see the following error, since the example config only has one configuration definition which
can't work on Windows.

.. code-block:: ipython3

    >>> import saspy
    >>> sas = saspy.SASsession()
    Using SAS Config named: default
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "C:\Users\sastpw\AppData\Local\Programs\Python\Python311\Lib\site-packages\saspy\sasbase.py", line 556, in __init__
        raise SASIONotSupportedError(self.sascfg.mode, alts=['IOM'])
    saspy.sasexceptions.SASIONotSupportedError: Cannot use STDIO I/O module on Windows. Try the following: IOM
    >>>

If you see that error, go read the configuration doc here: :doc:`configuration`


STDIO over SSH
--------------

The same issues in STDIO above are true here, with one extra component: ssh. The 'saspath' value has to be right, and
it has to be right on the remote Linux machine that you are ssh'ing to. That might not be the same path
as might be on your local SAS deployment.

Secondly, this requires that you have passwordless SSH configured and working for each user that will be connecting
between the local and remote machines, or or you can use sshpass. That can be diagnosed independant of this module and
Python. If the connection cannot be made, you should see that error message with the command that was
trying to be executed, and you can run it to get better diagnostic error messages that can tell you if its
a problem with your SSH credentials, the machine you're trying to reach isn't listening, or any other
problem there might be.

.. code-block:: ipython3

    >>> import saspy
    >>> sas = saspy.SASsession(cfgname='ssh')
    SAS Connection failed. No connection established. Double check your settings in sascfg_personal.py file.

    Attempted to run program /usr/bin/ssh with the following parameters:['/usr/bin/ssh', '-t', 'tom64-2', '/opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_en',
                                                                         '-fullstimer', '-nodms', '-stdio', '-terminal', '-nosyntaxcheck', '-pagesize', 'MAX', '']

    Try running the following command (where saspy is running) manually to see if you can get more information on what went wrong:
    /usr/bin/ssh -t tom64-2 /opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_en -fullstimer -nodms -stdio -terminal -nosyntaxcheck -pagesize MAX

    No SAS process attached. SAS process has terminated unexpectedly.

So, running that command can tell me what the problem is.

.. code-block:: ipython3

    Linux-1> /usr/bin/ssh -t Linux-2 /opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_en -fullstimer -nodms -stdio -terminal -nosyntaxcheck -pagesize MAX
    ssh: Could not resolve hostname Linux-2: Name or service not known

    or maybe another problem:

    Linux-1> /usr/bin/ssh -t Linux-2 /opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_en -fullstimer -nodms -stdio -terminal -nosyntaxcheck -pagesize MAX
    ssh: connect to host Linux-2 port 22: Connection refused

    or if it is that you do not have passwordless ssh set up, even though you can connect to that machine, you might see this (prompting you for pw)

    Linux-1> /usr/bin/ssh -t Linux-2 /opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_en -fullstimer -nodms -stdio -terminal -nosyntaxcheck -pagesize MAX
    user@Linux-2's password:
    Permission denied, please try again.
    user@Linux-2's password:
    Permission denied, please try again.
    user@Linux-2's password:
    Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).

To diagnose ssh further, you can ad -v (-vv, -vvv) to the command line to see more diagnostic information.
For instance, everything seems set up correctly but after running ssh it just says 'Connection closed by 10.17.12.14'


.. code-block:: ipython3

    Linux-1> /usr/bin/ssh -t Linux-2 /opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_en -fullstimer -nodms -stdio -terminal -nosyntaxcheck -pagesize MAX
    Connection closed by 10.17.12.14

    adding in -v[v[v]] can show that in this case, I just didn't have permission to get to this host via any authentication method

    Linux-1> /usr/bin/ssh -vvv -t Linux-2 /opt/sasinside/SASHome/SASFoundation/9.4/bin/sas_en -fullstimer -nodms -stdio -terminal -nosyntaxcheck -pagesize MAX
    [lot's of output removed. just showing some of it here]
    debug1: Host 'Linux-2' is known and matches the RSA host key.
    debug1: Found key in /usr/home/.ssh/known_hosts:480
    debug1: ssh_rsa_verify: signature correct
    debug1: SSH2_MSG_NEWKEYS sent
    debug1: expecting SSH2_MSG_NEWKEYS
    debug1: SSH2_MSG_NEWKEYS received
    debug1: SSH2_MSG_SERVICE_REQUEST sent
    debug1: SSH2_MSG_SERVICE_ACCEPT received
    debug1: Authentications that can continue: publickey,gssapi-keyex,gssapi-with-mic,password
    debug1: Next authentication method: gssapi-keyex
    debug1: No valid Key exchange context
    debug1: Next authentication method: gssapi-with-mic
    debug1: Unspecified GSS failure.  Minor code may provide more information
    Credentials cache file '/tmp/krb5cc_894' not found

    debug1: Unspecified GSS failure.  Minor code may provide more information
    Credentials cache file '/tmp/krb5cc_894' not found

    debug1: Next authentication method: publickey
    debug1: Trying private key: /usr/home/.ssh/identity
    debug1: Offering public key: /usr/home/.ssh/id_rsa
    debug1: Server accepts key: pkalg ssh-rsa blen 277
    debug1: read PEM private key done: type RSA
    Connection closed by 10.17.12.14


    As you can see, it tried multiple authentication methods, and although I have keys set up and the
    host is known, I just don't have a valid key for that system.


IOM
---

This access method has the most possibilities of having something misconfigured, because it has more
components that all have to connect together. But, it also has the most diagnostics to help you out.
There are basically two possibilities where something can go wrong: Java or IOM. Let's look at Java first.

There are three things that are likely to be the problem.

   1) Java isn't installed or configured right, or you don't have the right Java command for 'java' in your configuration definition.
   2) You don't have your classpath right, or don't have the right JAR files.
   3) An IOM specific issue like host/port aren't right, user/pw or Windows path issues


Java problems
^^^^^^^^^^^^^

This error is descibed below:

1) The system cannot find the file specified



Java startup problems will be caught and whatever system error(s) there were will be returned. And, like in the cases above,
you will still get the exact command trying to be run, so you can always run it too and see if there are any more diagnostics
and error messages.

Here an example of the first case, a bad path to the Java command. This example is from Jupyter Notebook on Windows.

.. code-block:: ipython3

    sas = saspy.SASsession(cfgname='winlocal', results='HTML', java='c:\java')

.. parsed-literal::

    The OS Error was:
    The system cannot find the file specified

    SAS Connection failed. No connection established. Double check your settings in sascfg_personal.py file.

    Attempted to run program c:\java with the following parameters:['c:\\java',
    '-classpath', 'C:\\java\\sas.svc.connection.jar;C:\\java\\log4j.jar;
    C:\\jars\\sas.security.sspi.jar;C:\\jars\\saspyiom.jar', 'pyiom.saspy2j',
    '-host', 'localhost', '-stdinport', '59110', '-stdoutport', '59111',
    '-stderrport', '59112', '-zero', '']

    If no OS Error above, try running the command (where saspy is running) manually to see what is wrong:

    c:\java -classpath "C:\java\sas.svc.connection.jar;C:\java\log4j.ja;C:\jars\sas.security.sspi.jar;C:\jars\saspyiom.jar" pyiom.saspy2j -host localhost -stdinport 59107 -stdoutport 59108 -stderrport 59109 -zero

    No SAS process attached. SAS process has terminated unexpectedly.

And if we submit that command, we get a slightly different error message than SASPy received, but it shows the same problem:
there is no c:\\java command to execute.

.. code-block:: ipython3

    C:> c:\java -classpath "C:\jars\sas.svc.connection.jar;C:\jars\log4j.jar;C:\jars\sas.security.sspi.jar;C:\jar\sas.core.jar;C:\jars\saspyiom.jar" pyiom.saspy2j -host localhost -stdinport 52061 -stdoutport 52062 -stderrport 52063 -zero
    'c:\java' is not recognized as an internal or external command, operable program or batch file.


Classpath problems
^^^^^^^^^^^^^^^^^^
So what about CLASSPATH problems? Here are three cases. The first is just the wrong path, so Java won't be able to find the main class to run.
The second case has a valid classpath, but is missing one of the IOM jars. The third is a case with EG versions of the jars, not from a SAS 9 install.

These errors are descibed below:

1) Error: Could not find or load main class pyiom.saspy2j
2) Error: Unable to initialize main class pyiom.saspy2j
   Caused by: java.lang.NoClassDefFoundError: org/omg/CORBA/UserException
3) Java Error:
   java.lang.NoClassDefFoundError: com/sas/services/connection/ConnectionFactoryException
4) The 'correct' error when your classpath is ok


1) Just what a bad classpath might look like:

.. code-block:: ipython3

    sas = saspy.SASsession(cfgname='winlocal', results='HTML', classpath='.')

.. parsed-literal::

    Java Error:
    Error: Could not find or load main class pyiom.saspy2j


    Subprocess failed to start. Double check your settings in sascfg_personal.py file.

    Attempted to run program java with the following parameters:['java', '-classpath',
    '.', 'pyiom.saspy2j', '-host', 'localhost', '-stdinport', '59102',
    '-stdoutport', '59103', '-stderrport', '59104', '-zero', '']

    If no Java Error above, try running the following command (where saspy is running) manually to see if it's a problem starting Java:
    java -classpath "." pyiom.saspy2j -host localhost -stdinport 59102 -stdoutport 59103 -stderrport 59104 -zero

    No SAS process attached. SAS process has terminated unexpectedly.


And if we submit that command, we see the same error that was reported.

.. code-block:: ipython3

    C:\>java -classpath "." pyiom.saspy2j -host localhost -stdinport 59102 -stdoutport 59103 -stderrport 59104 -zero
    Error: Could not find or load main class pyiom.saspy2j


To demonstate the error for a missing JAR file, let's comment out one of the IOM JAR files:

.. code-block:: ipython3

    cp  =  "C:\jars\sas.svc.connection.jar"
    cp += ";C:\jars\log4j.jar"
    cp += ";C:\jars\sas.security.sspi.jar"
    #cp += ";C:\jars\sas.core.jar"
    cp += ";C:\jars\saspyiom.jar"

    sas = saspy.SASsession(cfgname='winlocal', classpath=cp)

    Java Error:
    java.lang.NoClassDefFoundError: com/sas/util/ChainedExceptionInterface
            at java.lang.ClassLoader.defineClass1(Native Method)
            at java.lang.ClassLoader.defineClass(Unknown Source)
            at java.security.SecureClassLoader.defineClass(Unknown Source)
            at java.net.URLClassLoader.defineClass(Unknown Source)
            at java.net.URLClassLoader.access$100(Unknown Source)
            at java.net.URLClassLoader$1.run(Unknown Source)
            at java.net.URLClassLoader$1.run(Unknown Source)
            at java.security.AccessController.doPrivileged(Native Method)
            at java.net.URLClassLoader.findClass(Unknown Source)
            at java.lang.ClassLoader.loadClass(Unknown Source)
            at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
            at java.lang.ClassLoader.loadClass(Unknown Source)
            at java.lang.Class.getDeclaredMethods0(Native Method)
            at java.lang.Class.privateGetDeclaredMethods(Unknown Source)
            at java.lang.Class.privateGetMethodRecursive(Unknown Source)
            at java.lang.Class.getMethod0(Unknown Source)
            at java.lang.Class.getMethod(Unknown Source)
            at sun.launcher.LauncherHelper.validateMainClass(Unknown Source)
            at sun.launcher.LauncherHelper.checkAndLoadMain(Unknown Source)
    Caused by: java.lang.ClassNotFoundException: com.sas.util.ChainedExceptionInterface
            at java.net.URLClassLoader.findClass(Unknown Source)
            at java.lang.ClassLoader.loadClass(Unknown Source)
            at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
            at java.lang.ClassLoader.loadClass(Unknown Source)
            ... 19 more
    Error: A JNI error has occurred, please check your installation and try again
    Exception in thread "main"

    Subprocess failed to start. Double check your settings in sascfg_personal.py file.

    Attempted to run program java with the following parameters:['java', '-classpath', 'C:\\java\\sas.svc.connection.jar;C:\\java\\log4j.jar;C:\\jars\\sas.security.sspi.jar;C:\\jars\\saspyiom.jar',
    'pyiom.saspy2j', '-host', 'localhost', '-stdinport', '59110', '-stdoutport', '59111', '-stderrport', '59112', '-zero', '']

    If no Java Error above, try running the following command (where saspy is running) manually to see if it's a problem starting Java:
    java -classpath "C:\java\sas.svc.connection.jar;C:\java\log4j.jar;C:\jars\sas.security.sspi.jar;C:\jars\saspyiom.jar" pyiom.saspy2j -host localhost -stdinport 59110 -stdoutport 59111 -stderrport 59112 -zero

    No SAS process attached. SAS process has terminated unexpectedly.

And if we run that command ourselves... Same error as was reported.

.. code-block:: ipython3

    C:\> java -classpath "C:\java\sas.svc.connection.jar;C:\java\log4j.jar;C:\jars\sas.security.sspi.jar;C:\jars\saspyiom.jar" pyiom.saspy2j -host localhost -stdinport 59110 -stdoutport 59111 -stderrport 59112 -zero

    Error: A JNI error has occurred, please check your installation and try again
    Exception in thread "main" java.lang.NoClassDefFoundError: com/sas/util/ChainedExceptionInterface
            at java.lang.ClassLoader.defineClass1(Native Method)
            at java.lang.ClassLoader.defineClass(Unknown Source)
            at java.security.SecureClassLoader.defineClass(Unknown Source)
            at java.net.URLClassLoader.defineClass(Unknown Source)
            at java.net.URLClassLoader.access$100(Unknown Source)
            at java.net.URLClassLoader$1.run(Unknown Source)
            at java.net.URLClassLoader$1.run(Unknown Source)
            at java.security.AccessController.doPrivileged(Native Method)
            at java.net.URLClassLoader.findClass(Unknown Source)
            at java.lang.ClassLoader.loadClass(Unknown Source)
            at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
            at java.lang.ClassLoader.loadClass(Unknown Source)
            at java.lang.Class.getDeclaredMethods0(Native Method)
            at java.lang.Class.privateGetDeclaredMethods(Unknown Source)
            at java.lang.Class.privateGetMethodRecursive(Unknown Source)
            at java.lang.Class.getMethod0(Unknown Source)
            at java.lang.Class.getMethod(Unknown Source)
            at sun.launcher.LauncherHelper.validateMainClass(Unknown Source)
            at sun.launcher.LauncherHelper.checkAndLoadMain(Unknown Source)
    Caused by: java.lang.ClassNotFoundException: com.sas.util.ChainedExceptionInterface
            at java.net.URLClassLoader.findClass(Unknown Source)
            at java.lang.ClassLoader.loadClass(Unknown Source)
            at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
            at java.lang.ClassLoader.loadClass(Unknown Source)
            ... 19 more


2) The problem with versions 9 Java, not having CORBA available.

This problem has now been solved in a different way. I'll leave the original below for reference, but here is the
solution to this problem, along with Java 10, 11 ... which was not able to be solved that way as they no longer
even had CORBA available. As of version 3.1.1, saspy now includes the following 5 jars in the java/thirdparty directory.
Just add these jars into your classpath (as shown in the onfiguration doc for IOM) and this will work for any Java
version.

glassfish-corba-internal-api.jar
glassfish-corba-omgapi.jar
glassfish-corba-orb.jar
NOTICE.md
pfl-basic.jar
pfl-tf.jar


Old answer, no longer the solution:
A new issue has been reported when using Java9. The java IOM client is dependant on CORBA, which is in Java9 but no longer in its default search path.
This can be resolved by adding it back in, using the 'javaparms' key of your configuration definition as shown below.
Version 11 Java doesn't even ship CORBA, so the Java IOM client won't yet work with that version. The IOM group is currently investigating a solution to this.

If you see an error like this:

.. code-block:: ipython3

    Using SAS Config named: winiomwin
    Java Error:
    Error: Unable to initialize main class pyiom.saspy2j
    Caused by: java.lang.NoClassDefFoundError: org/omg/CORBA/UserException

Then you can add CORBA back into the search path via the 'javaparms' key (there may be other ways you can do this, but this has been reported to work):

.. code-block:: ipython3

    winiomwin = {"java"     : "java",
                 "encoding" : "windows-1252",
                 "classpath": cpW,
                 "javaparms": ["--add-modules=java.corba"],
                }




3) There can be an issue using some versions of these jars that aren't from a SAS9 instalation. Some EG client jars don't have all the necessary classes in them.

Although I don't have any of those jars to run an actual test for this, I've copied the following traceback from one of the issues where this was reported.
The NoClassDefFounfError is the clue here, referring to a com/sas/... class that isn't defined.


.. code-block:: ipython3

    Java Error:
    java.lang.NoClassDefFoundError: com/sas/services/connection/ConnectionFactoryException
    at java.lang.Class.getDeclaredMethods0(Native Method)
    at java.lang.Class.privateGetDeclaredMethods(Unknown Source)
    at java.lang.Class.privateGetMethodRecursive(Unknown Source)
    at java.lang.Class.getMethod0(Unknown Source)
    at java.lang.Class.getMethod(Unknown Source)
    at sun.launcher.LauncherHelper.validateMainClass(Unknown Source)
    at sun.launcher.LauncherHelper.checkAndLoadMain(Unknown Source)
    Caused by: java.lang.ClassNotFoundException: com.sas.services.connection.ConnectionFactoryException
    at java.net.URLClassLoader.findClass(Unknown Source)
    at java.lang.ClassLoader.loadClass(Unknown Source)
    at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
    at java.lang.ClassLoader.loadClass(Unknown Source)
    ... 7 more
    Error: A JNI error has occurred, please check your installation and try again
    Exception in thread "main"


4) When you run the java command yourself and the classpath is ok, you still get an error.

If you run the Java command and you see an error similar to the following, about a socket connection failure, that suggests that your CLASSPATH is correct
and that the problem might be connecting to the IOM server. That error shows that java came up and is running code from saspyiom.jar. It is trying to connect
back to the python process, which isn't running, thus the connection error. But it means, at least, saspyiom.jar was found and the other SAS jars too.

.. code-block:: ipython3

    java.net.ConnectException: Connection refused
            at java.net.PlainSocketImpl.socketConnect(Native Method)
            at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
            at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
            at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
            at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
            at java.net.Socket.connect(Socket.java:589)
            at java.net.Socket.connect(Socket.java:538)
            at java.net.Socket.<init>(Socket.java:434)
            at java.net.Socket.<init>(Socket.java:211)
            at pyiom.saspy2j.main(saspy2j.java:109)
    Exception in thread "main" java.lang.NullPointerException
            at pyiom.saspy2j.main(saspy2j.java:116)


IOM specific errors
^^^^^^^^^^^^^^^^^^^

So if Java is coming up, but you still fail to connect, then it is a problem connecting to IOM.
The IOM Error message will be reported, followed by the command that was trying to be run. Below
are the usual IOM errors and what to do about them.

There are a few obvious misconfigurations that can happen here, and these are the likely error you may see.
Scroll down based upon the number to see an example of that error and help on why that may occur and what to do about it.

1) **The application could not log on to the server "host:port". No server is available at that port on that machine.**


2) **The application could not log on to the server "host:port". The user ID "wrong_user" or the password is incorrect.**


3) **The native implementation module for the security package could not be found in the path.**


4) **The application could not find a command to launch a SAS Workspace Server.**


5) **The application could not log on to the server. The server process did not start.**


6) **The application could not log on to the server "localhost:0".  Integrated Windows authentication failed.**         OR

   **The security package failed while authenticating a user.**


7) **The application could not create a tunnel to the server "127.0.0.1:55517".**


8) **None of the requested encryption algorithms are supported by both peers: xxx.**


9) **An exception was thrown during the encryption key exchange.**


Here are examples of each of the above problems:


1) The application could not log on to the server "Linux-1:333". No server is available at that port on that machine.

   For this error, either the 'iomhost' or 'iomport' you've specified aren't right, or the server isn't up and available to be connected to.
   You may have specified the host or port for the metadata server instead of the host of the object spawner and
   port for theworkspace server.

.. code-block:: ipython3

    >>> sas = saspy.SASsession(iomport=333) # clearly the wrong port

    The application could not log on to the server "Linux-1:333". No server is available at that port on that machine.
    SAS process has terminated unexpectedly. Pid State= (11195, 64000)
    SAS Connection failed. No connection established. Double check your settings in sascfg_personal.py file.

    Attempted to run program /usr/bin/java with the following parameters:['/usr/bin/java', '-classpath', '/jars/sas.svc.connection.jar:/jars/log4j.jar:/jars/sas.security.sspi.jar:/jars/sas.core,jar:
    /jars/saspyiom.jar', 'pyiom.saspy2j', '-host', 'localhost', '-stdinport', '45757', '-stdoutport', '57809', '-stderrport', '33153', '-iomhost', 'Linux-1', '-iomport', '333', '-user', 'user', '']

    No SAS process attached. SAS process has terminated unexpectedly.


2) The application could not log on to the server "Linux-1:8591". The user ID "wrong_user" or the password is incorrect.

   Your credentials were specifed wrong, or you don't have permission to connect. This can also happen when there are more than one
   App Server and you didn't specify which one to connect to. The object spawner will only try the first one in its list, so it might
   be trying to connect you to the wrong App Serever.

.. code-block:: ipython3


    >>> sas = saspy.SASsession(omruser='wrong_user')

    The application could not log on to the server "Linux-1:8591". The user ID "wrong_user" or the password is incorrect.
    SAS process has terminated unexpectedly. Pid State= (11449, 64000)
    SAS Connection failed. No connection established. Double check your settings in sascfg_personal.py file.

    Attempted to run program /usr/bin/java with the following parameters:['/usr/bin/java', '-classpath', '/jars/sas.svc.connection.jar:/jars/log4j.jar:/jars/sas.security.sspi.jar:/jars/sas.core,jar:
    /jars/saspyiom.jar', 'pyiom.saspy2j', '-host', 'localhost', '-stdinport', '49660', '-stdoutport', '46794', '-stderrport', '51907', '-iomhost', 'Linux-1', '-iomport', '8591', '-user', 'wrong_user', '']

    No SAS process attached. SAS process has terminated unexpectedly.


3)  The native implementation module for the security package could not be found in the path.

    For Windows Local connection (and remote connections using IWA via {'sspi' : True}), you don't have the path to the sspiauth.dll in yout System Path variable. See the configuration doc
    to see how to specify this: https://sassoftware.github.io/saspy/install.html#local

.. code-block:: ipython3


    >>> import saspy
    >>> sas = saspy.SASsession()

    The native implementation module for the security package could not be found in the path.
    SAS process has terminated unexpectedly. RC from wait was: 4294967290
    SAS Connection failed. No connection established. Double check your settings in sascfg_personal.py file.

    Attempted to run program java with the following parameters:['java', '-classpath', 'C:\\java\\sas.svc.connection.jar;C:\\java\\log4j.jar;C:\\jars\\sas.security.sspi.jar;C:\\jars\\saspyiom.jar',
    'pyiom.saspy2j', '-host', 'localhost', '-stdinport', '59110', '-stdoutport', '59111', '-stderrport', '59112', '-zero', '']

    Be sure the path to sspiauth.dll is in your System PATH

    No SAS process attached. SAS process has terminated unexpectedly.


4) The application could not find a command to launch a SAS Workspace Server.

   For Windows Local connection, the registry doesn't have the right path to the SAS start up command.

.. code-block:: ipython3


    >>> sas = saspy.SASsession()
    The application could not find a command to launch a SAS Workspace Server.
    SAS process has terminated unexpectedly. RC from wait was: 4294967290
    SAS Connection failed. No connection established. Double check your settings in sascfg_personal.py file.

    Attempted to run program java with the following parameters:['java', '-classpath', 'C:\\java\\sas.svc.connection.jar;C:\\java\\log4j.jar;C:\\jars\\sas.security.sspi.jar;C:\\jars\\saspyiom.jar',
    'pyiom.saspy2j', '-host', 'localhost', '-stdinport', '59110', '-stdoutport', '59111', '-stderrport', '59112', '-zero', '']

    Be sure the path to sspiauth.dll is in your System PATH

    No SAS process attached. SAS process has terminated unexpectedly.


If you get this error: The application could not find a command to launch a SAS Workspace Server.
There is a workaround you can use. Oh course, having a clean SAS install should keep this from happening, but this error has been reported a couple times.
The work around for this is to use the 'javaparms' option on the configuration definition to specify the command manually as follows (use the right path on your system, of course):

.. code-block:: ipython3

    'javaparms' : ['-Dcom.sas.iom.orb.brg.zeroConfigWorkspaceServer.sascmd="C:\PROGRA~1\SASHome\SASFOU~1\9.4\SAS.EXE"']



5) The application could not log on to the server. The server process did not start.

   For Windows Local connection, the start up command in the registry isn't formated just right. Blanks, quotes, other.

.. code-block:: ipython3

    >>> sas = saspy.SASsession()
    The application could not log on to the server. The server process did not start.
    SAS process has terminated unexpectedly. RC from wait was: 4294967290
    SAS Connection failed. No connection established. Double check your settings in sascfg_personal.py file.

    Attempted to run program java with the following parameters:['java', '-Dcom.sas.iom.orb.brg.zeroConfigWorkspaceServer.sascmd=C:\\PROGRA~1\\SASHome\\SASFOU~1\\9.4\\SAS.EXE -config C:\\PROGRA~1\\SASHome\\SASFOU~1\\9.4\\sasv9.cfg -objectserver
    -nologo -noinal -noprngetlist', '-classpath',
    'C:\\Program Files\\SASHome\\SASDeploymentManager\\9.4\\products\\deploywiz__94485__prt__xx__sp0__1\\deploywiz\\sas.svc.connection.jar;
    C:\\Program Files\\SASHome\\SASDeploymentManager\\9.4\\products\\deploywiz__94485___xx__sp0__1\\deploywiz\\log4j.jar;
    C:\\Program Files\\SASHome\\SASDeploymentManager\\9.4\\products\\deploywiz__94485__prt__xx__sp0__1\\deploywiz\\sas.security.sspi.jar;
    C:\\Program Files\\SASHome\\SASDeploymentManager\\9.4\\products\\deploywiz__94485__pxx__sp0__1\\deploywiz\\sas.core.jar;
    C:\\ProgramData\\Anaconda3\\Lib\\site-packages\\saspy\\java\\saspyiom.jar',
    'pyiom.saspy2j', '-host', 'localhost', '-stdinport', '57425', '-stdoutport', '57426', '-stderrport', '57427', '-zero', '']









    Be sure the path to sspiauth.dll is in your System PATH

    No SAS process attached. SAS process has terminated unexpectedly.


If you get this error: The application could not log on to the server. The server process did not start.
And you have what seems to be the correct start up command in your registry; key=HKEY_CLASSES_ROOT\CLSID\{440196D4-90F0-11D0-9F41-00A024BB830C}\LocalServer32.
It may still not be formatted exactly right regaring quoted paths, or blanks in the paths, or the char8 '~' parts.
There is a easy way to have SAS re-register this in the Windows Registry that should clean this up and make it correct.
Run your sas.exe (you can do this from a CMD Prompt; may need fully qualified path for sas.exe) with the following option: /regserver

.. code-block:: ipython3

    [C:\...\]sas.exe /regserver


If this doesn't fix the issue, you can try the same workaround as #4 above, using the javaparms to specify the command.
The best option is to quote all paths in that command. In the error message above, you can see that javaparms was used to specify the command,
which failed. If I quote both of the paths in that parameter, then it works.

.. code-block:: ipython3

    '-Dcom.sas.iom.orb.brg.zeroConfigWorkspaceServer.sascmd="C:\\PROGRA~1\\SASHome\\SASFOU~1\\9.4\\SAS.EXE" -config "C:\\PROGRA~1\\SASHome\\SASFOU~1\\9.4\\sasv9.cfg" -objectserver -nologo -noinal -noprngetlist'


6) The application could not log on to the server "localhost:0".  Integrated Windows authentication failed.        OR

   The security package failed while authenticating a user.

   These errors imply that your hosts file doesn't have 'localhost' set as an alias for ip 127.0.0.1. Tech Support note
   55227 (http://support.sas.com/kb/55/227.html) identifies this issue.

.. code-block:: ipython3


    >>> sas = saspy.SASsession(cfgname='winlocal')

    We failed in getConnection
    The application could not log on to the server "localhost:0". Integrated Windows authentication failed.
    SAS process has terminated unexpectedly. RC from wait was: 4294967290
    SAS Connection failed. No connection established. Double check your settings in sascfg_personal.py file.

    Attempted to run program java with the following parameters:['java', '-classpath',
    'C:\\Program Files\\SASHome\\SASDeploymentManager\\9.4\\products\\deploywiz__94508__prt__xx__sp0__1\\deploywiz\\sas.svc.connection.jar;
    C:\\Program Files\\SASHome\\SASDepentManager\\9.4\\products\\deploywiz__94508__prt__xx__sp0__1\\deploywiz\\log4j.jar;
    C:\\Program Files\\SASHome\\SASDeploymentManager\\9.4\\products\\deploywiz__94508__prt__xx__sp0__1\\deploywiz\\sas.security.sspi.jar;
    C:\\Program Files\\SASHome\\SASDeplotManager\\9.4\\products\\deploywiz__94508__prt__xx__sp0__1\\deploywiz\\sas.core.jar;
    C:\\ProgramData\\Anaconda3\\Lib\\site-packages\\saspy\\java\\saspyiom.jar;
    C:\\Program Files\\SASHome\\SASVersionedJarRepository\\eclipse\\plugins\\sas.rutil_904500.0.0.0816190000_v940m5\\sas.rutil.jar;
    C:\\Program Files\\SASHome\\SASVersionedJarRepository\\eclipse\\plugins\\sas.rutil.nls_904500.0.0.20170816190000_v940m5\\sas.rutil.nls.jar;
    C:\\Program Files\\SASHome\\SASVersionedJarRepository\\eclipse\\plugins\\sastpj.l_6.1.0.0_SAS_20121211183517\\sastpj.rutil.jar',
    'pyiom.saspy2j', '-host', '127.0.0.1', '-stdinport', '49207', '-stdoutport', '49208', '-stderrport', '49209', '-zero', '-lrecl', '1048576', '']

    Be sure the path to sspiauth.dll is in your System PATH

    No SAS process attached. SAS process has terminated unexpectedly.


7) The application could not create a tunnel to the server "127.0.0.1:55517"  (the port number will vary)

   This is an error that can occur trying to make a Local IOM connection to local SAS on your Windows machine.
   This one was recently reported in issue 354. Turns out it's a problem with having the SAS_IOM_PROXYLIST environment variable set.
   This is needed sometimes for connecting to remote IOM servers when there are firewalls in the way, or something about that. Either
   way, it's not needed and can cause a failure trying to use a local SAS install over IOM Local saspy connection. Luckilly it's easy
   to resolve. You can simply unset that variable in your Python session before trying to get your SASsession. You can do this inline
   if you need this set for remote IOM connections from saspy, or put it in your sascfg_personal.py file if you only connect to local
   SAS from saspy but need the variable set for other applications that need it.

.. code-block:: ipython3

    >>> import os
    >>> os.environ['SAS_IOM_PROXYLIST']='https://www.sas.com'
    >>> os.environ['SAS_IOM_PROXYLIST']
    'https://www.sas.com'
    >>>
    >>> import saspy; sas = saspy.SASsession(cfgname='winlocal'); sas
    We failed in getConnection
    The application could not create a tunnel to the server "127.0.0.1:54243".
    SAS process has terminated unexpectedly. RC from wait was: 4294967290


    >>> # unset the variable here now. You can do this in your config file it you need it all the time
    >>> del(os.environ['SAS_IOM_PROXYLIST'])

    >>> os.environ['SAS_IOM_PROXYLIST']
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "C:\ProgramData\Anaconda3\lib\os.py", line 669, in __getitem__
        raise KeyError(key) from None
    KeyError: 'SAS_IOM_PROXYLIST'
    >>> # it's been unset now
    >>>
    >>> import saspy; sas = saspy.SASsession(cfgname='winlocal'); sas
    SAS Connection established. Subprocess id is 37100

    Access Method         = IOM
    SAS Config name       = winlocal
    SAS Config file       = C:\ProgramData\Anaconda3\lib\site-packages\saspy\sascfg_personal.py
    WORK Path             = C:\Users\sastpw\AppData\Local\Temp\SAS Temporary Files\_TD17980_d10a626_\Prc2\
    SAS Version           = 9.04.01M5P09132017
    SASPy Version         = 3.6.2
    Teach me SAS          = False
    Batch                 = False
    Results               = Pandas
    SAS Session Encoding  = wlatin1
    Python Encoding value = cp1252
    SAS process Pid value = 17980
    >>>


8) None of the requested encryption algorithms are supported by both peers: xxx. (xxx is the method; AES)

   This error identifies that your Workspace server is configured to use encryption. The specific method may
   show up at the end of the message; for instance AES. This means that you don't have the 3 encryption jars
   in the iomclient directory of the saspy install. See the configuration section for IOM regarding this:
   https://sassoftware.github.io/saspy/configuration.html#attn-as-of-saspy-version-3-3-3-the-classpath-is-no-longer-required

   If this error is for AES, then there's another solution than having to get those 3 jars and add them to the deployment,
   but only for SAS versions prior to M7. M7 requires the encryption jars.
   Java 8 (release greater than 151), has the needed support for this in it. So you just need to install the current Java 8
   or higher to solve this without needing the jars.


9) An exception was thrown during the encryption key exchange.

   This is another possible error having to do with encryption, and is addressed by adding the 3 encryption jars to your
   saspy deployment, as identified in number 8 just above.


So, hopefully this has shown you how to diagnose connection and configuration problems. When you have things set up right, you shouldn't
have any problems, it should just work!


*********************
Problems running code
*********************


My model didn't run
-------------------

When you run an analytical method there are a number of things that occur.
The goal is to have informative messages when things go wrong and in this section we'll explain what his happening
and how to check the various stages

#. Are the required parameters included?
   Each analytical method has a set of required parameters (there are a few that have an empty set but
   all the methods must have this specified).

   The simplest way to find the required and optional parameters is to use the `'?'` functionality.

   Here are two examples for the forest and hplogistic methods respectively:

   ::

       ?ml.forest()
       ?stat.hplogistic()

   The next best option is to use the :doc:`api` for the given method
   Both ways will show you the set of required parameters and then the list of optional parameters. The requred set
   and optional make up the complete set of parameters the method will take.

   If you are missing required parameters you will receive a SyntaxError and processing will stop.

   .. parsed-literal::

       SyntaxError: You are missing 1 required statements:
       {'model'}

   Missing optional parameter will produce no warning.

   .. note:: The `data` parameter does not appear in the required set but it is required for all modeling methods.

#. Do you have extra parameters?
   If you include parameters that are neither required *or* optional then will be removed but as a best practice
   don't test the system.

#. Are the parameter the correct type?
   Parameters must be specified with the correct type. If you provided an invalid type you should recieve a
   SyntaxWarning or SyntaxError and processing will stop.

   Here are a few of the most common parametes and their valid types

   * The `data` parameter must be a :any:`SASdata` object.

   * The `model` parameter is a str.

   * The `target` and `inputs` can be str, list, or dict types.

   * The `nominals` must be a list type.

   Making the parameters handle more types is a great way to get involved. Enter an issue and we can help you.

#. Were errors generated during execution?
   If you make it this far, the error is probably in the running of the genereated SAS code.
   To investigate, you can display the ERROR_LOG attribute on your :any:`SASresults` object.

   ::

       rf_model.ERROR_LOG

   The resulting output will the SAS log for that generated code. You will be able to see the SAS syntax and then
   and error or warning messages in context.

   If the ERROR_LOG doesn't give you enough information to resolve your issue you can execute the
   following code (assuming your session object is named sas).

   ::

       print(sas.saslog())

   This will output the SAS log for then entire session (since you last restarted the kernel or your initial connection).

