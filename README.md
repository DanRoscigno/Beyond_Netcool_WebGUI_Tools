# Beyond_Netcool_WebGUI_Tools
The basic tool types don't always get you where you need to go.  Sometimes you need form validation, or you want to pre-populate form items programmatically.  Maybe you want to run selects against ObjectServer tables.  Use some CGI, someHTML, and some REST calls to the ObjectServer to get the job done.  This is a simple example thatsolves a real problem for my customer.

Let's start at the end, here is what I built
============================================
![Initiate SWAT HTML Form](https://user-images.githubusercontent.com/25182304/33861720-5ff6633a-dead-11e7-834a-73fd7a504a76.png)

Using an HTML form and the OMNIbus REST API via CGI gave me:
 - A place to give the user instructions
 - The ability to pre-populate the time in GMT (I found that users routinely used local time, which messes up my SLA record keeping)
 
 ![GMT Detail](https://user-images.githubusercontent.com/25182304/33849888-0178fc78-de80-11e7-9f14-c077e8b80b4d.png)
 - The ability to have a non-selectable default choice in a drop down list (I found that with the default tools I ended up paging out the wrong DevOps teams very often because the user requesting the page out would accept the default.  This is actually the issue that pushed me to switch from the default Web GUI tools as the wrong DevOps team was getting woken up.)
 
 ![Dropdown List Detail](https://user-images.githubusercontent.com/25182304/33861871-04caf9ca-deae-11e7-9558-c09d917a5cda.png)

 - The REST API connects via the omni.dat / interfaces file, which means it connects to a virtual pair without writing in your own failover /failback routine

HTML forms need a place to live
===============================

If you are using the DASH/Jazz for Service Management web server that comes with WebSphere Application Server you can add HTML files to the "myBox" directory. Here is a quick lesson:

 - Become the user that owns the Jazz files (netcool in my case)

        su - netcool

 - CD into the myBox dir

        find /opt/IBM -name myBox
        cd ... wherever.../JazzSM/ui/myBox/

 - Edit your html files in myBox/web_files/

 - Deploy your files Note: enclose the smadmin password in double quotes and escape those special chars

        ./deployMyBox.sh -username smadmin -password "C\$tfood"

 - This is then the URL: https://1.2.3.4:16311/myBox/yourfile.html

 - For example, my filename is testing.html, so: https://1.2.3.4:16311/myBox/testing.html


CGI files already have a home
=============================

Pop your file in place in the standard cgi-bin dir (on my system it is /opt/IBM/netcool/gui/omnibus_webgui/etc/cgi-bin; just try *find /opt/IBM -name cgi-bin*), then register it with Web GUI as always (ping me if you need a howto)

Where are we?
=============

- We have a place to put HTML files (*myBox*)
- We have a place to put CGI files (*standard Web GUI cgi-bin dir*)
- We have a simple HTML form (testing.html is in this git repo)

Questions?
==========

Writing to the ObjectServer
===========================
The HTML form that we have been looking at is used by a person to report an issue (think about customer care, they receive a call and typically open a ticket.  In my case they are filling out a form that posts **(note the term *POST* rather than *insert* as I am using the REST API and therefore REST terminology)** an event into the ObjectServer rather than creating a ticket.  In the past, people would often call nco_sql to insert an event from a CGI, or use a Perl lib, or Sybase Jconnect.  I have used all of the above, but I prefer to use the REST API, so let's take a look at that from a Python CGI.  The full script is in this repo at InitiateSWATTEST.cgi

Before we dive in and look at all of the details, let's trace one bit of information.  In the HTML form one of the form components is the drop down list of products (Product A SaaS, B, C, etc.)

Tracing the Impacted service information
----------------------------------------

 - In the HTML form I defined a drop down listing the services we support.  When the user selects one and submits the form the key *Service* gets passed over with a value of whatever service I selected in the drop down list
![HTML Form Snippet](https://user-images.githubusercontent.com/25182304/33856716-0216709a-de97-11e7-958f-577c248055a1.png)

 - Over in the CGI I read the contents of the environment and parse the QUERY_STRING, which contains the information from the HTML form
![Reading the Form data](https://user-images.githubusercontent.com/25182304/33856718-0225a4c0-de97-11e7-9cae-801614ae202f.png)

 - Next (still in the CGI), I specify the ObjecServer fields that I will write to when I POST.  I specify a field named *application* here as a string.  In my schema, the application is the service name.
![Specifying the ObjectServer fields to write](https://user-images.githubusercontent.com/25182304/33856720-02426eb6-de97-11e7-8d5e-d9061b17be4e.png)

 - And then, still in the CGI I use the contents of the variable Service in the POST.
![Setting the content of the applicationfield](https://user-images.githubusercontent.com/25182304/33856721-024f0676-de97-11e7-997e-40adba569fe1.png)

**Let's look at:**
 - Authentication
 - Receiving data from the HTML form
 - A REST ObjectServer POST
 
 Authentication
 --------------
 I am out of the habit of including usernames and passwords in my code as I commit my code to places like GitHub.  I keep the secrets in config files that do not leave my server, in the code I put a comment with the format of the config file so that I have a record of how to do it the next time.  If you use Python I suggest that you look at ConfigParser.
 ![ConfigParser for auth](https://user-images.githubusercontent.com/25182304/33853621-8905d1f0-de8c-11e7-94f5-2e3675c3001f.png)
 
 Receiving data from the HTML form
 ---------------------------------
 The QUERY STRING is standard CGI, and is the same as we have been doing in OMNIbus CGI tools for15 years.  Python provides some nice ways to traverse the string.
 ![Parsing the HTML form data](https://user-images.githubusercontent.com/25182304/33853625-8dbb3690-de8c-11e7-9bf1-65464451e365.png)
 
 A REST ObjectServer POST
 --------------------------
 The format of the POST is intuitive, specify the names and types of the fields, and then the content of the row(s) that you want to POST.
 ![REST POST](https://user-images.githubusercontent.com/25182304/33853650-acbf9cc0-de8c-11e7-880e-48ab9292e6a5.png)
 
 Here are examples from the <a href="https://ibm.co/2AdBodY" target="_blank">REST API documentation</a> 
 
Now where are we?
=================
- Once the user submits the form the CGI POSTs a new row to the ObjectServer
- A trigger then sends a POST to Alert Notification, which alerts the Site Reliability Engineering (SRE) folks
- An SRE then:
    - creates a Slack channel
    - pages the DevOps team

What could be done to streamline this?
=================
- Once the user submits the form the CGI
   - POSTs a new channel to the Slack Team
   - POSTs a new row to the ObjectServer with the form info and Slack channel
   - POSTs an alert to Alert Notification for the DevOps team to join the Slack channel

