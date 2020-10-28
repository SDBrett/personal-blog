---
author: Brett Johnson
categories:
- Python
date: "2016-05-22T18:01:49Z"
id: 282
title: Checking when external IP has changed with Python
url: /BrettsITBlog/2016/05/checking-when-external-ip-has-changed-with-python/
---
This is an exercise I thought of on the drive home from work. Just a task to help give context to the concepts I have been learning.

This exercise came from wanting to VPN into my home network. The home network has a dynamic external IP address, which means I need a way to know if that IP has changed.

The script IPcheck.py helps solve that problem. It';s been created to check the current external IP against a stored value and notify by email if the two do not match. It';s intended to be run as a scheduled task. For example, I have a cron job setup on my Raspberry Pi.

The GitHub for this project is located [here](https://github.com/oversizedspoon/IPCheck). At the time of writing this entry, there is still a little work to be done, such as adding doc strings.

The script was written for Python3 and requires the following modules:

  * requests
  * os.path
  * smtplib
  * email.mime.text

Before running the script, the file needs to have some values set to work.

The first three are required for emailing to work. For my testing, I didn';t need to auth against the SMTP server, as such this is not in the script. If you require SMTP auth then it will need to be added. _ipstorepath_ is set as a raw string to allow paths such as "C:\&#8221; without the need for escape characters.

As a bit of a side note, when using cron, I needed to use a full path in _ipstorepath_, however when running manually, a filename is all that';s needed.

{{< highlight python >}}smtpserver = "Enter SMTP Server"
fromaddress = "Enter From Email Address"
toaddress = "Enter Destination Email Address"
ipstorepath = r"IPstore.txt"{{< / highlight >}}

&nbsp;

The logic of the script is as follows:

  * Get external IP address
  * Check if IPStore file exists 
      * If it does, compare the stored value to the retrieve value 
          * If the values do not match, update the IPStore file and email the new IP
          * If the values match, end
  * If the file does not exist, create the file with the retrieved IP

The code for this logic:

{{< highlight python >}}if os.path.isfile(ipstorepath): #test if IP Store file exists
    currentip = open(ipstorepath, 'r+') #Open IP Store file for reading
    if currentip.read() != ipaddr: #Check if IP Address received is the same what is stored in IP Store file. Code below will run if currentip does not equal IP address received from IPIFY
        currentip.close()
        updateipstore(ipaddr, ipstorepath)
        sendnewip(ipaddr, toaddress, fromaddress, smtpserver) #Send email with new IP Address
else: #Runs if IP Store file doesn't exist
    updateipstore(ipaddr, ipstorepath) #Used to create IP store file{{< / highlight >}}

&nbsp;

The following code is used to get the current external IP Address. This snippet will grab the external IP address from ipify as plain text.

{{< highlight python >}}ipaddr = requests.get("https://api.ipify.org").text{{< / highlight >}}

&nbsp;

The following function is used to write the retrieved IP address to the IPStore file.

{{< highlight python >}}def updateipstore(ipaddr, ipstorepath):
    #Writes Current IP Address to the IP Store file
    currentip = open(ipstorepath, 'w')
    currentip.write(ipaddr)
    currentip.truncate()
    currentip.close(){{< / highlight >}}

&nbsp;

Sending emails from Python requires a few core arguments. These have been set at the top of the script. In addition to those we need to past _ipaddr_ to the function so it can be contained email. We use MIMEText to ensure the email is properly formatted.

{{< highlight python >}}def sendnewip(ipaddr, toaddress, fromaddress, smtpserver):
    #Module for sending emails
    import smtplib
    from email.mime.text import MIMEText
    text = "Your new IP is: " + str(ipaddr)
    msg = MIMEText(text, 'plain')
    msg['From'] = fromaddress
    msg['To'] = toaddress
    msg['Subject'] = "IP Address Change"
    server = smtplib.SMTP(smtpserver, 25)
    server.starttls()
    text = msg.as_string()
    server.sendmail(fromaddress, toaddress, text)
    server.quit(){{< / highlight >}}

That about covers this little script. I intend to create more scripts like this to help develop my skills and provide a breakdown explaining the logic and code segments.

&nbsp;