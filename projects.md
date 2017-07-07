---
layout: page
title: Projects
---

Some of the blogs, code, and research I've released or collaborated on over the years.
 * Mar. 2017 - [Pass-the-Hash is Dead: Long Live LocalAccountTokenFilterPolicy](http://www.harmj0y.net/blog/redteaming/pass-the-hash-is-dead-long-live-localaccounttokenfilterpolicy/)
 * Feb. 2017 - [Loading .NET assemblies via SQL](http://sekirkity.com/command-execution-in-sql-server-via-fileless-clr-based-custom-stored-procedure/) - Wrote the original code that inspired this post. Wrote a shellcode injector for this in C# after reading through the PowerUpSQL code and seeing a comment referring to it. Original use case was to get around command-line logging and heavy monitoring of common lateral spread techniques.
 * Jan. 2017 - [Roasting AS-REPs](http://www.harmj0y.net/blog/activedirectory/roasting-as-reps/)/[S4U2Pwnage](http://www.harmj0y.net/blog/activedirectory/s4u2pwnage/) - Figuring out how Kerberos pre-authentication and S4U2Self/S4U2Proxy work and how attackers can leverage/abuse them. 
 * Jul. 2016 - [KeeThief](https://github.com/HarmJ0y/KeeThief/) - [How dump KeePass databases](http://www.harmj0y.net/blog/redteaming/keethief-a-case-study-in-attacking-keepass-part-2/) in post-exploitation scenarios. I wrote most of the memory scraping and decryption shell code. 
 * Dec. 2015 - [The Evolution of Offensive PowerShell Invocation](https://silentbreaksecurity.com/powershell-jobs-without-powershell-exe/) - Executing PowerShell in multiple .NET AppDomains for parallel script execution and to prevent .NET type collisions.
 * Dec. 2014 - [UnmanagedPowerShell](https://www.github.com/leechristensen/UnmanagedPowerShell) - Executing PowerShell from unmanaged code.  This code formed the basis of PowerShell capabilities in [Empire](https://github.com/EmpireProject/Empire), [Cobalt Strike](https://blog.cobaltstrike.com/2016/05/18/cobalt-strike-3-3-now-with-less-powershell-exe/), Silent Break Security's Slingshot, and Meterpreter<sup>[1](http://buffered.io/posts/continued-meterpreter-development/)</sup><sup>[2](http://www.darkoperator.com/blog/2016/4/2/meterpreter-new-windows-powershell-extension)</sup>.
