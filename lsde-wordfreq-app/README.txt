=========================================================================
| University of Bristol - EMATM0051 (LSDE) Coursework                   |
| WordFreq App - Overview and Setup Instructions                        |
| Original App (c) AWS Inc.                                             |
| Additional shell scripts and documentation (c) Alan Forsyth, UoB 2021 |
| tl18303@bristol.ac.uk                                                 |
| Doc v1.20.                                                            |
=========================================================================

Overview
--------
Word Frequency is a sample service built with AWS SDK for Go.
The service takes advantage of Amazon EC2 service, Amazon Simple Storage service, Amazon Simple Queue Service and Amazon DynamoDB to collect and report the top 10 most common words of a text file.
The original sample app released by AWS is here:
https://github.com/aws-samples/aws-go-wordfreq-sample

Setup guide
-----------
The rest of this document covers step by step setup of the standard application, ready for further work required in the LSDE coursework (see the separate coursework documentation).

Pre-requisites
--------------
- AWS Academy Foundation Learner Lab account, registered and activated
- SSH Terminal / PuTTy or similar on Windows
- Internet Browser - Google Chrome or Firefox are best


Task A - Launching the Development Instance
-------------------------------------------

1. Log in to AWS Academy and start up the Learner Lab

2. On the lab screen note the remaining credits, and check this as you start each session.
-- $300 should be more than enough for the coursework and all testing (you will need under $50)
-- If you are down to $20 or under shut down any EC2 instances and contact the instructor for advice.

3. Note the session time. You have 4 hours before you need to refresh this Lab page and reopen any AWS Console pages.
-- All resources you create will remain in your Learner Lab for as long as you have credit available, they are not destroyed at the end of a session!

4. Click 'AWS' with green button to open a new AWS Console page.

5. Go to the EC2 service and launch an instance with the following non-default settings:
-- AMI: 
--- Go to Community AMIs
--- Copy/paste in this AMI ID: ami-05e4673d4a28889fe (and press ENTER)
--- Select 'Cloud9Ubuntu-2021-10-28T13-33' image
-- Instance Type: t2.micro
-- IAM Role: EMR_EC2_DefaultRole
-- Tags: Key=Name Value=wordfreq-dev
-- Security Group: name: wordfreq-dev-sg rule: SSH (default rule)
-- KeyPair: name: learnerlab-keypair  (download the .pem file and keep it safe!)


Task B - Create the S3 Bucket
-----------------------------

1. Select S3 using the Services dropdown at the top left of the EC2 Console page (it can be helpful to open this in a new browser tab).

2. Create a new S3 Bucket with the following non-default settings:
-- Bucket name: a unique name, using alphanumeric characters or dashes, perhaps using your initials or date;
   e.g. af-wordfreq-nov20

3. Make a note of your bucket name for later.


Task C - Create the SQS Queues
------------------------------

1. Select SQS using the Services dropdown, ideally opening in another new browser tab.

2. Create a new SQS queue (for file processing jobs) with the following non-default settings:
-- Queue type: Standard
-- Queue name: wordfreq-jobs
-- Access policy: Advanced
-- Change the JSON policy code section that looks like this...:

"Principal": {
        "AWS": "<12 digits>"
      },

-- ...to the following (this allows any AWS entity to write to the queue, not just the queue owner):

"Principal": {
        "AWS": "*"
      },

3. Once the queue is created, take a note of the queue URL ('https://sqs.us-east-1.amazonaws.com/....').

4. Create a second SQS queue (for file processing results) with the following non-default settings:
-- Queue name: wordfreq-results
-- Access policy: Advanced [configure as for the jobs queue]

5. Once again, make a note of the queue URL once it's created.


Task D - Configure the File Upload notification from Bucket to Queue
--------------------------------------------------------------------

1. Return to the S3 Console page and click on your Bucket > Properties.

2. Scroll down to 'Event notifications', click 'Create event notification'.

3. Configure the following non-default settings:
-- Event name: file-upload-ev
-- Event types: [Select 'All object create events']
-- Destination: SQS queue
-- SQS queue: [Select 'wordfreq-jobs']


Task E - Log in to the Dev Instance
-----------------------------------

1. Return to the EC2 Console and select the wordfreq-dev instance (select the checkbox).

2. Click the Connect button above and select the 'SSH client' tab.

3. The connection instructions are correct if your PC is running Linux or MacOS (in a Terminal window):
-- If you are connecting from a Windows PC, following instructions in sections 'Prerequisites' and 'Connecting to your Linux instance' on the following page:
   https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html
   (This is for PuTTy - other SSH clients are available)
   There is also a helpful 4-min video available from Linux Academy (ensure you use the 'ubuntu' username when logging in):
   https://www.youtube.com/watch?v=bi7ow5NGC-U&lc=Ugh4DAc2SqJj3HgCoAEC

   NOTE: If using PuTTy, pasting text from the system clipboard onto the command line is often achieved using a mouse right-click only.

4. When logging in for the first time, you may need to confirm the connection is valid by typing 'yes'.

5. You should see a command line prompt of the form: 'ubuntu@ip-172-XXX-XXX-XXX' (it may take 30 seconds to finally display) - you have logged in successfully.

NOTE: To quit the SSH session later, type: exit


Task F - Copy the Application Code zip onto the Dev Instance
------------------------------------------------------------

1. Return to the S3 Console page and click on your Bucket name.

2. Click 'Upload' and select the coursework zip file ('lsde-wordfreq-app.zip'), OR drag the coursework zip file directly onto this webpage and confirm.

3. Once uploaded, click on the blue zip file link now present in the 'Files and Folders' section.

4. On the file details page, select the file, copy the 'S3 URI' and note it down - we will use this to access the file from the CLI;
e.g. s3://af-wordfreq-nov20/lsde-wordfreq-app.zip

5. Return to your SSH CLI window and run the following command to update the system (it may take a minute or two):

sudo apt update

6. Type the following 'S3 list' command to ensure you can see your S3 Bucket name, which shows you have correct permissions:

aws s3 ls

7. Run the following command, entering your noted down 'S3 URI' instead of S3_URI (don't forget the final dot, indicating to copy to the current directory):

aws s3 cp S3_URI .

8. Check you now have the zip file downloaded on your Dev Instance, then unzip the package:

ls
unzip lsde-wordfreq-app.zip

9. Run the 'ls' command again - you should now see you have a new 'lsde-wordfreq-app' directory.

NOTE: We will leave the zip file in case you need to completely delete the directory AND all its files and start again.
      You can do this with the following command in the home directory: rm -rf lsde-wordfreq-app; unzip lsde-wordfreq-app.zip


Task G - Set up and Configure the WordFreq App
----------------------------------------------

1. Change directory to the app folder and ensure all shell scripts have correct execution permissions:

cd lsde-wordfreq-app
chmod +x *.sh

2. Run the 'setup.sh' script, which will install the GO language runtime and any dependencies, as well as creating the DynamoDB 'wordfreq' database table:

./setup.sh

NOTE: This script should take a couple of minutes to run, and end with 'Setup complete.'. If errors are shown, run it again.
      If there are still other errors (ignore the 'table already exists' error), and you don't get 'Setup complete', please add a post on the BB forum or book an Office Hours session.

3. Optional: In another browser tab, open the DynamoDB Console, click Tables, select wordfreq and click View Items, which will display items (rows) added to the table.

4. You will now need to manually edit the 'run_worker.sh' and 'run_upload.sh' scripts to refer to the correct SQS queue URLs.
-- These instructions assume you will use the 'Vi' editor, but you can install and use any other one, such as GEdit or EMACS, as required.
-- Type the following to open and edit the run_worker.sh script in Vi:

vi run_worker.sh

-- Using the arrow keys, move the cursor down to the following line:

export WORKER_QUEUE_URL="https://sqs.us-east-1.amazonaws.com/12345678/wordfreq-jobs"

-- Press 'i' to enter Insert mode in Vi, then delete the URL and paste or type in your noted URL for the SQS jobs queue between the quotes.
-- Similarly edit the following line, updating the URL value for the results queue with your own:

export WORKER_RESULT_QUEUE_URL="https://sqs.us-east-1.amazonaws.com/12345678/wordfreq-results"

-- Now exit Insert mode by pressing the Escape key (Esc)
-- Enter the following key strokes to write updates and quit: colon, lower case 'w' and lower case 'q', then press ENTER, i.e.

:wq

-- You should now be back to the normal command prompt. Perform the same update to the run_upload.sh script:

vi run_upload.sh


Task H - Test the Worker and Upload functionality
-------------------------------------------------

1. We will now try to test the basic app functionality. We first need to empty the jobs queue of any spurious messages.
-- Return to the SQS Console window and select the 'wordfreq-jobs' queue (click on the radio button on the left)
-- Click on the 'Purge' button and type in 'purge' where required to confirm you want to delete all messages.

2. You will need two SSH Terminal sessions for the 'worker' and 'upload' processes.
-- Open a second Terminal or PuTTy window and log in again with the same SSH command / PuTTy profile as earlier.

3. Window 1: Worker
-- Ensure you are in the correct directory and start the worker process. If there are errors, check your SQS URLs in the run_worker.sh file.

cd ~/lsde-wordfreq-app
./run_worker.sh

-- You should see some lines of log output, which will increase when the worker finds jobs to process.

NOTE: The main WordFreq process is this 'worker' application, which runs continuously, checking for jobs on the queue and processing them.

4. Window 2: Uploader
-- Ensure you are in the correct directory and run the run_upload.sh script, filling in your S3 bucket name instead of S3_BUCKET as below.

NOTE: The run_upload.sh script will upload a test file to the S3 bucket and wait for results:

cd ~/lsde-wordfreq-app
./run_upload.sh S3_BUCKET assets/test_file_01.txt

-- Switch between the two windows as these processes are running.
-- You should see after a few moments that the upload script confirms the file was uploaded, the worker retrieves the job, and the upload script displays results shortly afterwards.

NOTE: The worker performance has been deliberately crippled by adding an extra 'wait' of 10 seconds during processing, which you must not modify.
This makes it much easier to ensure the scaling operation is effective without requiring hundreds of input files.

-- If you observed the output described above, the basic application is working. We now just need to set it up as a service.


Task I - Setting up the Worker Service
--------------------------------------

1. In Window 1:
-- Press CTRL+C to exit the worker.sh process.
-- Set up the WordFreq Worker service by running the shell script:

./configure-service.sh

NOTE 1: This command installs the Worker.sh command as a service, which runs in the background and will auto-start on boot.
        It's important that any auto-scaling EC2 worker instances have this service configured in this way.

NOTE 2: If you do NOT get a 'Service started successfully' message, run again, or run through the setup process again.
        If you still experience issues, please post on the BlackBoard Discussion Forum for LSDE, or book an Office Hours session.

-- To view the output logs from the running wordfreq worker service, enter the following (CTRL+C to exit):

sudo journalctl -f -u wordfreq

NOTE: To stop or start the wordfreq worker service, run the following commands, respectively:

sudo systemctl stop wordfreq
sudo systemctl start wordfreq

2. In Window 2:
-- Press the 'up' key to run the last run_upload.sh command as in Task 8, or type it in again with your S3 Bucket specified:

./run_upload.sh S3_BUCKET assets/test_file_01.txt

3. Check again that Window 1 shows the worker output in the log entries, and Window 2 displays the final results.

4. At this point, press CTRL+C in Window 1, close Window 2 (type 'exit') if you don't need it anymore, and pat yourself on the back, we're done!
-- BUT ... make an AMI backup of this EC2 instance - see 'strong recommendation' below - THEN you can relax. ;-)


Task J - Consult the Coursework Doc for tasks
---------------------------------------------

- When implementing and testing autoscaling, you will be mainly using the following operations of those we have learned here:
-- Uploading files to S3 for processing
-- Purging (emptying) messages from the queues
-- Reviewing the worker logs on an EC2 instance
-- Stopping or starting the workfreq worker service if necessary on an EC2 instance
-- Running the run_upload.sh script on the Dev Instance for testing (but you can also upload text files directly to S3 for load testing, etc.)

NOTE: The infrastructure configuration we have performed is functional, but not necessarily optimal or best practice...


IMPORTANT NOTES
===============
- When you have finished a coursework session, ensure that any EC2 instances are stopped to minimise cost.
NOTE 1: The approximate cost for a running EC2 t2.micro instance is about 2 cents (US) per hour, but it adds up if never stopped.
NOTE 2: You do not pay for stopped EC2 instances, but you still pay for their EBS storage volumes, however this is a fraction of the EC2 cost.
- When you restart an EC2 instance, the free Public IP changes, so if you are accessing SSH via the IP address only, you will need to copy the new one.

------------------------------------------------------------------------
STRONG Recommendation: Create an AMI Backup before ending this session!!
------------------------------------------------------------------------
- Create an AMI image from the running EC2 instance (Instances > select wordfreq-dev > Actions > 'Image and templates' > Create image).
- If you lose your configured EC2 instance, check with the instructors on how to retrieve your configuration from the AMI, or rebuild a new EC2 instance as above.
- EBS Snapshots can also be used to store incremental backups of an EBS disk volume used by an instance.


SUPPORT
=======
For any general or technical issues with this setup, please start a new post on the BlackBoard Coursework Discussion Forum:
<Blackboard URL>

Alternatively, for one-to-one support, please book an Office Hours Session with the instructor or a TA:
https://outlook.office365.com/owa/calendar/ematlsdeofficehours@bristol.ac.uk/bookings/
