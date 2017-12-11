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

pop your file in place, then register it with Web GUI as always (ping me if you need a howto)
