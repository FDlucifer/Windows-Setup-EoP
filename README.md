# **CVE-2020-1571 Windows Setup Elevation of Privileges Bypass**

**Summary:**

Let&#39;s check out what did Microsoft about this vulnerability

An elevation of privilege vulnerability exists in Windows Setup in the way it handles permissions.

A locally authenticated attacker could run arbitrary code with elevated system privileges. After successfully exploiting the vulnerability, an attacker could then install programs; view, change, or delete data; or create new accounts with full user rights.

The security update addresses the vulnerability by ensuring the Windows Setup properly handles permissions.

I was the founder of the vulnerability.

The vulnerability exist when the windows setup doesn't properly enforce permission on C:\\$WINDOWS.~BT folder creation allowing a local user to execute arbitrary code with system privileges.

Let&#39;s take a look on windowsupdatebox.exe before the security patch

![GitHub Logo](/images/image1.png)

As you can see there's a CreateDirectory (as I highlighted in red) which is written in the following form

CreateDirectoryW(FileName, 0i64)

0i64 -> NULL

FileName -> C:\\$WINDOWS.~BT

Actually creating a directory with null security descriptor in c:\\ allow authenticated users to have read&write&delete access on the directory because of the default inherit permission from c:\\

However after few code (the blue highlighted ones) there&#39;s a call of SetSecurityInfo which change the security descriptor of the directory, it&#39;s clearly a race condition, a user will attempt to set a reparse point to an arbitrary location after the CreateDirectoryW and before the CreateFile call. That&#39;s a short description for CVE-2020-1571.

Let&#39;s try to do some damage, let&#39;s take a look at the patched function

![GitHub Logo](/images/image2.png)

Seems look like there&#39;s no SetSecurityInfo call anymore but instead they pass the security descriptor to the function directly which prevent abuse or a possible race condition.

Interesting but there&#39;s still something we can look for, let&#39;s drive deeply in the code, let&#39;s find operation before this function was called maybe we can find something we can abuse.

The caller actually do some initialization before trying to create this directory,

![GitHub Logo](/images/image3.png)

I was looking for something until I saw this and I can smell the vulnerability here, and after taking a look I found something. It seem look like that the clean-up process attempt to delete C:\\$WINDOWS.~BT before attempting to create the directory again without properly checking for reparse point, it&#39;s probably an abuse able point for arbitrary file deletion. Let's look what procmon gonna say

![GitHub Logo](/images/image4.png)

And as it should be there's no SetSecurityFile, but what if the directory already exist ? as we seen in the code it must deleted

![GitHub Logo](/images/image5.png)

And as you can see here WindowsUpdateBox is trying to delete the directory with the entire subcontent, it seems look like there's a check for reparse point but it&#39;s not really enough to prevent abuse, if we put an oplock and then switched a reparse point we will have a TOCTOU.

The technique used here is the same one used in the windows defender zero day bug which I disclosed a month ago [https://0x00sec.org/t/windows-defender-av-zero-day-vulnerability/22258](https://0x00sec.org/t/windows-defender-av-zero-day-vulnerability/22258)

It took from me a day to find &amp; write poc &amp; write this write-up.

NOTE: The bug is only exploitable on windows 10, on any version of lower than 2004.

Steps To Reproduce:

1. Run the poc
2. Go to settings->updates->Feature update to windows 10, version X
![GitHub Logo](/images/image6.png)
3. A SYSTEM shell should be spawned

PoC Video:
<iframe width="1280" height="720" src="https://www.youtube.com/embed/rRxfZQ0jPLA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
