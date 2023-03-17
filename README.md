# Google-Data-Fusion-ExitOnOutOfMemoryError-with-input-of-1.4GB

For a Data Engineer, as I claim to be, it should be a simple task: transform a file from **xml** to **json**.

There are plenty of library in almost every language that do it in one single command line. Easy. However, the task is much harder when working with huge files, which do not fit in the RAM of your laptop. Let me describe step-by-step how I run through the limits of Data Fusion.

Here is the logic:

I built an xml file
I created a bucket on Google Cloud Storage for both the input and output files
I designed the pipeline on Data Fusion, than deployed and run it
I checked the output stored on Google cloud Storage
I repeated the 4 steps of above with different xml files: the first one was a small file of only 14 Kbytes, to test the pipeline. Than I replicated the same content of the xml file in order to generate bigger files. I tested the pipeline with different file sizes, and it worked up to 141 MBytes, but when I tried with a file of 1.4 GBytes the Datafusion pipeline generated an error.
Following the files used for the tests:

OriginalOrderFile.xml: https://drive.google.com/file/d/1OYJw700VxMSZJnF-PVP580NY98lt6ouL/view?usp=share_link

Header_OriginalOrderFile.xml:

<?xml version="1.0" encoding="utf-8"?>  
<Root xmlns="http://www.adventure-works.com">  
  <Orders>
Body_OriginalOrderFile.xml: https://drive.google.com/file/d/1KfOL7_RfWOsB5Pgu8zsHPHJkGiBMhcAa/view?usp=share_link

Footer_OriginalOrderFile.xml:

  </Orders>  
</Root>
Starting from the Header, Body and Footer, I was able to build a bigger xml file by replicating the body and attaching the header and the footer at the end of the process. The script used for generating the bigger files is the following:

rm OriginalOrderFile_costruito.xml;k=1;cat Header_OriginalOrderFile.xml >> OriginalOrderFile_costruito.xml; while [ $k -le 10 ]; do cat Body_OriginalOrderFile.xml >> OriginalOrderFile_costruito.xml; k=$(( $k + 1 )); done ; cat Footer_OriginalOrderFile.xml >> OriginalOrderFile_costruito.xml
Increasing the value of $k parameter you can get bigger xml files, as following:

k=10 generates a 141 KB file
k=100 generates a 1.4 MB file
k=1000 generates a 14.1 MB file
k=10000 generates a 141 MB file
k=100000 generates a 1.4 GB file I launched this script directly on the shell of the Google Cloud Platform console. Than, I copied the file to the Google Cloud Storage using the following command:
gsutil cp OriginalOrderFile_costruito.xml gs://<your bucket>/OriginalOrderFile_costruito.xml
Finally I created the pipeline on Google Cloud Data Fusion. Using the console: Data Fusion => Create an Instances => Instance name: "xml-to-json" => Edition: Basic => Create

In the meanwhile, download the pipeline configuration file from here: https://drive.google.com/file/d/1j329sOzbyBbX6c65vna-V5Sx_E9518zN/view?usp=share_link

Once the instance is created (it can take about 10 minutes), set the autoscaling feature of the Data Fusion: click on "System Admin" => "Configuration" => "System Compute Profiles" => chose "Autoscaling Dataproc", up to 84 cores

Now, you can upload the pipeline file in the GCP console: Data Fusion => Instances => click on view instance => menu => list => click on the button with the plus symbol on the green background which is on the top right of the screen => click in import in the pipeline block => chose the downloaded pipeline configuration file. If it give a versioning error, click on the "Fix All" button on the bottom right of the screen.

In the pipeline, click on modify to change the source and the output files paths (change the field "Path") in the GCP Storage.

When you run using the xml files up to 141 MB, the pipeline succeed in only 8 minutes.

When you run it using the 1.4 GB xml file, the Data Fusion gives an error.

Here is the detailed log downloaded by the Data Fusion console, clicking on the "Download All => View Raw logs" button: https://drive.google.com/file/d/1Jhrv9xkBXZBb_1VI3iGavNGlOHc6TDNo/view?usp=share_link

Hence, the autoscaling feature of Data Flow is not really working or is not enough performing for a transformation of a file of 1.4 GB!!!

Can anybody help me? How can I run the Data Fusion pipeline for bigger file, let's say 140 GB file?
