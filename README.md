# Windows-Setup-EoP
Summary:
Let’s check out what did Microsoft about this vulnerability
An elevation of privilege vulnerability exists in Windows Setup in the way it handles permissions.
A locally authenticated attacker could run arbitrary code with elevated system privileges. After successfully exploiting the vulnerability, an attacker could then install programs; view, change, or delete data; or create new accounts with full user rights.
The security update addresses the vulnerability by ensuring the Windows Setup properly handles permissions.
Description:
I was the founder of the vulnerability.
The vulnerability exist when the windows setup doesn’t properly enforce permission on C:\ $WINDOWS.~BT folder creation allowing a local user to execute arbitrary code with system privileges.
Let’s take a look on windowsupdatebox.exe before the security patch.
