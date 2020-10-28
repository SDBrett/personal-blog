---
author: Brett Johnson
categories:
- PowerShell
date: "2017-03-29T12:57:56Z"
tags:
- Microsoft
- PowerShell
title: 'PowerShell: Logging in JSON format'
url: /BrettsITBlog/2017/03/powershell-logging-json-format/
---

When writing scripts for clients, it';s important to generate good logs. It';s a common position where a script works perfectly in test. But in production something is askew.

Being able to look back at the process and step through after the fact is essential.

Usually my logs are written in CSV format. CSV is great for parsing. But, it';s awful to run in a text file. Due to this, I decided to modify my log function. It now writes in JSON format.

#### The Design

The function is intended to be run as part of a script. As tasks complete in the script, they call this function to provide data. This data is then written to the file.

Each time the function runs, it has to reopen the log file and pull the data. This could create a performance impact on very large scripts. Still, due to the require of accurate logs for debug. It is necessary to get that data to a non-volatile storage medium, rather then store in RAM.

For large scripts, you may need to consider multiple log files. Implement log levels or what needs to log.

Named parameters were used for script readability.

#### The Code

I think I could do better with naming the function, but that';s stuck at 2 AM.

{{< highlight powershell >}}
Function Write-Verbose-Log
    {
        param
        (
            [Parameter(Mandatory=$True)][string]$Group,
            [string]$Message,
            [string]$Exception,
			[string]$Result,
			[string]$LogPath
        )
    $Time_Stamp = ($Time_Stamp = Get-Date).Datetime
  
	$NewLogData  = @{"Time" = $Time_stamp; "Message" = $Message; "Exception" = $Exception; "Result" = $Result}
    
	#Open Existing Log File to Get current data
	$CurrentLog = Get-Content -Path $LogPath -Raw | ConvertFrom-Json

	#If the log file is empty, CurrentLog cannot have a method called as it's 'Null Valued'.
	#Test for Null Value, if true, create CurrentLog as an Object 
    If($CurrentLog -eq $Null)
    {
        $CurrentLog = New-Object -TypeName PSObject
    }	
	
	#Group is used as the top level key value. Should be a unique identifier
	#The value for this key is the data provided into the function when called
    $CurrentLog | Add-Member -Type NoteProperty -Name $Group -Value $NewLogData

    $CurrentLog | ConvertTo-Json | Out-File $LogPath
}{{< / highlight >}}

My most recent use for the function, was working with AD groups. As a result I wanted to use the group name as the top level key name. Being the key value, it is a mandatory parameter.

Values which are "Null&#8221; will write the string "&#8221; to the log file.

The function and it';s parameters can be modified easily, making it applicable.

#### Output and Parsing

Below is a sample output I have run. You will notice that the exception message uses escaped characters. This is part of the ConvertTo-JSON function. When using ConvertFrom-JSON these will be properly encoded to the original character.

{{< highlight json >}}
{
    "Group1":  {
                   "Time":  "Wednesday, 29 March 2017 11:13:45 AM",
                   "Exception":  "",
                   "Message":  "Group Found",
                   "Result":  "Found"
               },
    "Group2":  {
                   "Time":  "Wednesday, 29 March 2017 11:14:09 AM",
                   "Exception":  "Cannot bind argument to parameter \u0027a\u0027 because it is an empty string.",
                   "Message":  "Group Not Found",
                   "Result":  "Error"
               }
}{{< / highlight >}}

Using unique key values as we have, presents an interesting challenge for parsing. If we wanted to find groups which had an error. We need to search by the nested key &#8216;Result';.

{{< highlight powershell >}}Function Parse-JSONLog
{
	Param
	(
	[Parameter(Mandatory=$True)][string]$LogPath #Path to log file
	)

	$LogContent = Get-Content $LogPath | ConvertFrom-Json
	$Errors = @()
    $Found = @()

	ForEach ($KeyGroup in $LogContent.psobject.Properties)
	{
        #Check if the Key has a result of "Error"
		if ($KeyGroup.psobject.Properties.value.result -eq "Error")
		{
			$Errors += $KeyGroup.name	
		}
        #Check if the Key has a result of "Error"
		Elseif ($KeyGroup.psobject.Properties.value.result -eq "Found")
		{
			$Found += $KeyGroup.name  
		}
	}
    #Combine Error and Found Arrays
    $Results = @{"ErrorGroups" = $errors; "FoundGroups" = $Found}
    return $Results
}{{< / highlight >}}

Run

{{< highlight text >}}$x = Parse-JSONLog -LogPath .\test.txt{{< / highlight >}}

And you can use $x.ErrorGroups to see of the Groups which had errors.

&nbsp;