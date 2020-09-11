# Getting Started with Cloud Storage and Cloud SQL

# Overview
In this lab, you create a Cloud Storage bucket and place an image in it. You'll also configure an application running in Compute Engine to use a database managed by Cloud SQL. For this lab, you will configure a web server with PHP, a web development environment that is the basis for popular blogging software. Outside this lab, you will use analogous techniques to configure these packages.

You also configure the web server to reference the image in the Cloud Storage bucket.

# Objectives
In this lab, you learn how to perform the following tasks:

- Create a Cloud Storage bucket and place an image into it.

- Create a Cloud SQL instance and configure it.

- Connect to the Cloud SQL instance from a web server.

- Use the image in the Cloud Storage bucket on a web page.

# Task 1: Sign in to the Google Cloud Platform (GCP) Console

login into Google Cloud using SDK using the command below

    gcloud auth login

select your project with the command below

    gcloud config set project PROJECT_ID



# Task 2: Deploy a web server VM instance

the following command lines will create a VM instance of name bloghost, machine type default that is n1-standard-1, with Debian image, in the zone us-central1-a and given http as tag name for firewall rule, including a startup-script which install a simple aparche2 web server when the VM starts.

    gcloud beta compute instances create bloghost --zone=us-central1-a --machine-type=n1-standard-1 --metadata=startup-script=apt-get\ update$
    apt-get\ install\ apache2\ php\ php-mysql\ -y$'\n'service\ apache2\ restart --tags=http --image=debian-9-stretch-v20200805

the command set http firewall rule on the VM

    gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http



# Task 3: Create a Cloud Storage bucket 

For convenience, enter your chosen location into an environment variable called LOCATION. use this command

    export LOCATION=US

For convenience, enter your project ID into an environment variable called DEVSHELL_PROJECT_ID. use this  command

    export DEVSHELL_PROJECT_ID=<enter your project ID here>

you can use the "echo $" command to verify


Enter this command to make a bucket named after your project ID in the US loctaion:

    gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID

Retrieve a banner image from a publicly accessible Cloud Storage location:

    gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png

Copy the banner image to your newly created Cloud Storage bucket:

    gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png

Modify the Access Control List of the object you just created so that it is readable by everyone:

    gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png



# Task 4: Create the Cloud SQL instance

Create instance of name=blog-db with password=admin of version=MYSQL-5.7 in the zone=us-central1-a with the command below

    gcloud sql instances create blog-db --password="admin" --sql-version=MYSQL_5_7" --zone=us-central1-a

the command line below will ADD USER ACCOUNT of name=blogdbuser and password=password
, the “%” that refers to the host ip address that this user can interact with

    gcloud sql users create blogdbuser --instance=blog-db -i blog-db --host='%' --password='password'

configuration an authorized network connection between the SQL instance and the VM instance (bloghost) with the commands below. Be sure to use the external IP address of your VM instance followed by /32. Do not use the VM instance's internal IP address. Do not use the sample IP address shown here, Please note that you ip will be different.

    gcloud compute instances list

    gcloud sql instances patch blog-db --authorized-networks="34.66.179.46/32"



# Task 5: Configure an application in a Compute Engine instance to use Cloud SQL

SSH in the row for your VM instance bloghost using the SSH command below

    ssh bloghost

change your working directory to the document root of the web server:

    cd /var/www/html

Use the nano text editor to edit a file called index.php:

>>>> sudo nano index.php

Paste the content below into the file:

************************************************
<html>
<head><title>Welcome to my excellent blog</title></head>
<body>
<h1>Welcome to my excellent blog</h1>
<?php
 $dbserver = "CLOUDSQLIP";
$dbuser = "blogdbuser";
$dbpassword = "DBPASSWORD";
// In a production blog, we would not store the MySQL
// password in the document root. Instead, we would store it in a
// configuration file elsewhere on the web server VM instance.

$conn = new mysqli($dbserver, $dbuser, $dbpassword);

if (mysqli_connect_error()) {
        echo ("Database connection failed: " . mysqli_connect_error());
} else {
        echo ("Database connection succeeded.");
}
?>
</body></html>
**************************************************

Press Ctrl+O, and then press Enter to save your edited file.
Press Ctrl+X to exit the nano text editor.

Restart the web server:

    sudo service apache2 restart

Open a new web browser tab and paste into the address bar your bloghost VM instance's external IP address followed by /index.php. The URL will look like this: <35.192.208.2/index.php>

When you load the page, you will see that its content includes an error message beginning with the words:

`<Database connection failed: ...>`


Use the below command to get the IP address of the Sql instance and use that in the variable “$dbserver”:

    gcloud sql instances list

Return to your ssh session on bloghost. Use the nano text editor to edit index.php again.

    sudo nano index.php

In the nano text editor, replace CLOUDSQLIP and DBPASSWORD with the Cloud SQL instance Public IP address and with the Cloud SQL database password that you defined above respectively

safe and exit

Restart the web server:

    sudo service apache2 restart

Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, the following message appear <Database connection succeeded.>



# Task 6: Configure an application in a Compute Engine instance to use a Cloud Storage object

since your already Upload an image of name my-excellent-blog.png in your bucket gs://$DEVSHELL_PROJECT_ID

so here should be the path to th image  “https://storage.googleapis.con/<project_id>/<storage_item_name>”

so edith the index.php file again

    sudo nano index.php

Use the arrow keys to move the cursor to the line that contains the h1 element. Press Enter to open up a new, blank screen line, and then paste the URL you copied earlier into the line inside a <img src=""> tag as shown below

<img src="https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png">

Do not copy the URL shown here. Instead, copy the URL shown by the Storage browser in your own Cloud Platform project.

save and exit

Restart the web server:

    sudo service apache2 restart


Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, its content now includes a banner image.



# Congratulations!
In this lab, you configured a Cloud SQL instance and connected an application in a Compute Engine instance to it. You also worked with a Cloud Storage bucket.