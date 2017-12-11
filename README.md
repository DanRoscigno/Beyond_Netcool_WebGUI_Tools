# Beyond_Netcool_WebGUI_Tools
The basic tool types don't always get you where you need to go.  Sometimes you need form validation, or you want to pre-populate form items programatically.  Maybe you want to run selects against ObjectServer tables.  Use some CGI, someHTML, and some REST calls to the ObjectServer to get the job done.  This is a simple example thatsolves a real problem for my customer.

Let's start at the end, here is what I built
============================================
![Initiate SWAT HTML Form](https://user-images.githubusercontent.com/25182304/33849871-f4e6a5a0-de7f-11e7-9b8e-4b23b49dadb6.png)

The form above is nothing fancy, but are the things it gave me:
 - A place to give the user instructions
 - The ability to pre-populate the time in GMT (I found that users routinely used local time, which messes up my SLA record keeping)
 
 ![GMT Detail](https://user-images.githubusercontent.com/25182304/33849888-0178fc78-de80-11e7-9f14-c077e8b80b4d.png)
 - The ability to have a non-selectable default choice in a drop down list (I found that with the default tools I ended up paging out the wrong DevOps teams very often because the user requesting the page out would accept the default)
 
 ![Dropdown List Detail](https://user-images.githubusercontent.com/25182304/33849876-f8dab098-de7f-11e7-8d76-b16b3d7ad01b.png)


HTML forms need a place to live
===============================

If you are using the DASH/Jazz for Service Management web server that comes with WebSphere Application Server you can add HTML files to the "myBox" directory. Here is a quick lesson:

 - Become the user that owns the Jazz files (netcool in my case)

        su - netcool

 - CD into the myBox dir

        find /opt/IBM -name myBox
        cd ... wherever.../JazzSM/ui/myBox/

 - Edit your html files in myBox/web_files/

 - Deploy your files Note: enclose the smadmin password in doublequotes and escape those special chars

        ./deployMyBox.sh -username smadmin -password "C\$tfood"

 - This is then the URL: https://1.2.3.4:16311/myBox/yourfile.html

 - For example, my filename is initiateSWAT.html, so: https://1.2.3.4:16311/myBox/initiateSWAT.html


CGI files already have a home
=============================

Pop your file in place in the standard cgi-bin dir (on my system it is /opt/IBM/netcool/gui/omnibus_webgui/etc/cgi-bin; just try *find /opt/IBM -name cgi-bin*), then register it with Web GUI as always (ping me if you need a howto)

Where are we?
=============

We have a place to put HTML files (*myBox*)
We have a place to put CGI files (*standard Web GUI cgi-bin dir*)
We have a simple HTML form (testing.html is in this git repo)

Questions?
==========

Writing to the ObjectServer
===========================
The HTML form that we have been looking at is used by a person to report an issue (think about customer care, they receive a call and typically open a ticket.  In my case they are filling out a form that inserts an event into the ObjectServer rather than creating a ticket.  In the past, people would often call nco_sql to insert an event from a CGI, or use a Perl lib, or Sybase Jconnect.  I have used all of the above, but I prefer to use the REST API, so let's take a look at that from a Python CGI.  The full script is in this repo at InitiateSWATTEST.cgi

**Let's look at:**
 - Authentication
 - Receiving data from the HTML form
 - A REST ObjectServer insert
 
 Authentication
 --------------
 foo
 ![ConfigParser for auth](https://user-images.githubusercontent.com/25182304/33853621-8905d1f0-de8c-11e7-94f5-2e3675c3001f.png)
 
 Receiving data from the HTML form
 ---------------------------------
 foo
 ![Parsing the HTML form data](https://user-images.githubusercontent.com/25182304/33853625-8dbb3690-de8c-11e7-9bf1-65464451e365.png)
 
 A REST ObjectServer insert
 --------------------------
 foo
 ![REST insert](https://user-images.githubusercontent.com/25182304/33853650-acbf9cc0-de8c-11e7-880e-48ab9292e6a5.png)
 

