# *ir-rescue*

*ir-rescue* is composed of two sister scripts that collect a myriad of **forensic data** from 32-bit and 64-bit **Windows** systems (*ir-rescue-win*) and from **Unix** systems (*ir-rescue-nix*). The scripts respect the order of volatility and artifacts that are changed with the execution (*e.g.*, prefetch files on Windows) and are intended for **incident response** use at different stages in the analysis and investigation process. Each are described as follows:

* ***ir-rescue-win*** is fully written in Batch and can be set to perform comprehensive and customized acquisitions of specific types of *live data* and of *historical data* from available Volume Shadow Copy Service (VSS) copies. *ir-rescue-win* makes use of built-in Windows commands and well-known third party utilities from Sysinternals and NirSoft, for instance, some being open-source. PowerShell and the Windows Management Instrumentation (WMI) are not used in order to make *ir-rescue-win* transversally compatible.

* ***ir-rescue-nix*** is written in Bash (v4+) and makes use of built-in Unix commands. Some commands used might not be POSIX-compliant and therefore might not be available on some Unix-like systems or variants, especially on older operating systems.

*ir-rescue* is designed to group data collections according to data type. For example, all data that relates to networking, such as open file shares and Transmission Control Protocol (TCP) connections, is grouped together, while running processes, services and tasks are gathered under malware. The acquisition of data types and other general options are specified in a simple **configuration file**. It should be noted that the script launches a great number of commands and tools, thereby leaving a considerable **footprint** (*e.g.*, strings in the memory, prefetch files, program execution caches) on the system. The runtime varies depending on the computation power, disk write throughput and configurations set. Disk performance is especially important if secure deletion is set and when dumping 64-bit memory (usually 8 GB in size), which can take a considerable amount of time.

*ir-rescue* has been written for incident response and forensic analysts, as well as for security practitioners alike. It represents an effort to streamline host data collection, regardless of investigation needs, and to rely less on on-site support when remote access or live analysis is unavailable. It can thus be used to leverage the already bundled tools and commands during forensic activities. 

# Dependencies and Usage

*ir-rescue* relies on a number of third-party utilities for gathering specific data from hosts. The versions of the tools are listed in the last section and are provided with the package as is and, therefore, their licenses and user agreements must be accepted before running *ir-rescue*. Their descriptions and organization in the are given below, with both 32-bit and 64-bit versions of Windows tools included adjacently, if applicable:

* `tools-nix/`: third-party tools folder for *ir-rescue-nix*:
	* `ascii/`: text ASCII art files in `*.txt` format;
	* `cfg/`: configuration files:
		* `ir-rescue-nix.conf`: main configuration file for *ir-rescue-nix*;
		* `nonrecursive-(hidden|md5sum).txt`: hidden files and `md5sum` non-recursive locations;
		* `nonrecursive.txt`: non-recursive locations for multiple tools;
		* `recursive-(exec|hidden|md5sum).txt`: executables, hidden files and `md5sum` recursive locations;
		* `recursive.txt`: recursive locations for multiple tools;
	* `mem/`: memory tools:
		* `linpmem-2.1.post4` (64-bit ELF): dumps the memory;
* `tools-win\`: third-party tools folder for *ir-rescue-win*:
	* `ascii\`: text ASCII art files in `*.txt` format;
	* `cfg\`: configuration files:
		* `ir-rescue-win.conf`: main configuration file *ir-rescue-win*;
		* `nonrecursive-(acl|iconsext|md5deep).txt`: `accesschk[64].exe`, `iconsext.exe` and `md5deep[64].exe` non-recursive locations;
		* `nonrecursive.txt`: non-recursive locations for multiple tools;
		* `recursive-(acl|iconsext|md5deep).txt`: `accesschk[64].exe`, `iconsext.exe` and `md5deep[64].exe` recursive locations;
		* `recursive.txt`: recursive locations for multiple tools;
	* `cygwin\`: Cygwin tools and Dynamic Linked Libraries (DLLs):
		* `tr.exe`: used to cut out non-printable characters;
		* `grep.exe`: used to filter date with regular expressions;
	* `evt\`: Windows events tools:
		* `psloglist.exe`;
	* `fs\`: filesystem tools:
		* `tsk\`: The Sleuth Kit (TSK) tools and DLLs:
			* `fls.exe`: walks the Master File Table (MFT);
		* `AlternateStreamView[64].exe`: lists Alternate Data Streams (ADSs);
		* `ExtractUsnJrnl[64].exe`: extracts the `C:\$Extend\$UsnJrnl` (NTFS journal) file without the sparsed zeroes;
		* `md5deep[64].exe`: computes Message Digest 5 (MD5) hash values;
		* `ntfsinfo[64].exe`: shows information about NTFS;
		* `RawCopy[64].exe`: extracts data at the NTFS level;
	* `mal\`: malware tools:
		* `autoruns[64].exe`: dumps autorun locations to the autoruns binary format;
		* `autorunsc[64].exe`: lists autorun locations;
		* `densityscout[64].exe`: computes an entropy-based measure for detecting packers and encryptors;
		* `DriverView[64].exe`: lists loaded kernel drivers;
		* `handle[64].exe`: lists object handles;
		* `iconsext.exe`: extracts icons from Portable Executables (PEs);
		* `Listdlls[64].exe`: lists loaded DLLs;
		* `pslist[64].exe`: lists running processes;
		* `PsService[64].exe`: lists services;
		* `sigcheck[64].exe`: checks digital signatures within PEs;
		* `WinPrefetchView[64].exe`: displays the contents of prefetch files;
	* `mem\`: memory tools:
		* `winpmem_1.6.2.exe`: dumps the memory;
	* `misc\`: miscellaneous tools:
		* `LastActivityView.exe`: displays a timeline of recent system activity;
		* `OfficeIns[64].exe`: lists installed Microsoft Office add-ins;
		* `USBDeview[64].exe`: lists previously and currently connected USB devices;
	* `net\`: network tools:
		* `psfile[64].exe`: lists files opened remotely;
		* `tcpvcon.exe`: lists TCP connections and ports and UDP ports;
	* `sys\`: system tools:
		* `accesschk[64].exe`: lists user permissions of the specified locations;
		* `logonsessions[64].exe`: lists currently active logon sessions;
		* `PsGetsid[64].exe`: translates between Security Identifiers (SIDs) and user names and vice-versa;
		* `Psinfo[64].exe`: displays system software and hardware information;
		* `psloggedon[64].exe`: lists locally logged on users that have their profile in the registry;
	* `web\`: web tools:
		* `BrowsingHistoryView[64].exe`: lists browsing history from multiple browsers;
		* `ChromeCacheView.exe`: displays the Google Chrome cache;
		* `IECacheView.exe`: displays the Internet Explorer cache;
		* `MozillaCacheView.exe`: displays the Mozilla Firefox cache;
	* `yara\`: YARA tools and signatures:
		* `rules\`: `*.yar` rules folder;
		* `yara(32|64).exe`: YARA main executable;
		* `yarac(32|64).exe`: YARA rules compiler;
	* `7za.exe`: compresses files and folders;
	* `nircmdc[64].exe`: features extensive functionality, among of which taking screenshots;
	* `sdelete(32|64).exe`: securely deletes files and folders;
* `data\`: data folder created during runtime with the collected data:
	* `<HOSTNAME>-<DATE>\`: `<DATE>` follows the `YYYYMMDD` format:
		* `ir-rescue`\: folder for `ir-rescue`-related data
			* `ir-rescue.log`: verbose log file of status messages;
			* `screenshot-#`: numbered screenshots for *ir-rescue-win* only;
		* folders named according to the data type set for collection.

*ir-rescue-win* needs to be run under a command line console with **administrator rights** while *ir-rescue-nix* needs to be run under a command line window with **root privileges**. Both require no arguments and make use of a respective configuration file to set desired options. As such, executing the scripts simply needs the issuing of the files as follows:

* `ir-rescue-win-v1.x.y.bat`, or
* `ir-rescue-nix-v1.x.y.sh`.

Some tools that perform recursive searches or scans are set only to recurse on specific folders. This makes the data collection more targeted while taking into account run time performance as the folders specified are likely locations for analysis due to extensive use by malware. The locations for **recursive search** and **non-recursive search** for Windows and Unix systems can be changed at will in the respective text files under the configuration folders. Some of the tools have dedicated files with specific locations in which to and not to recurse. These are named `recursive-<tool>.txt` and `nonrecursive-<tool>.txt`, with `<tool>` being changed to the tool name. Each file must have one **location as full path** per line without trailing backslashes or forward slashes.

During runtime, all characters printed to the Standard Output (`STDOUT`) and Standard Error (`STDERR`) channels are logged to UTF-8 encoded text files. This means that the output of tools are stored in corresponding folders and text files. Status ASCII messages are still printed to the console in order to check the execution progress. A temporary folder created under `%TEMP%\ir-rescue` or `/tmp/ir-rescue` is used to store runtime data (*e.g.*, memory dump drivers and links to VSS copies) and is deleted upon completion. After collection, data can be compressed into a password-protected archive and accordingly deleted afterwards, if set to do so.

# Configuration File

The configuration file of each *ir-rescue-win* and *ir-rescue-nix* are composed of simple binary directives (`true` or `false`) for the general behaviour of the scripts, for which data types to collect and for which advanced tools to run. Lines preceded by a hash sign (`#`) are considered comments. These are used to briefly describe what each option does, to enumerate folders, files or registry keys important to provide some context, as well as to list relevant tools. The descriptions below applies only to *ir-rescue-win*, but they serve as an example to understand the overall approach and the configuration file of *ir-rescue-nix*.

For *ir-rescue-win*, data is grouped into the types given by the following directives:

* `memory`: this options sets the collection of the memory;
* `registry`: this option sets the collection of system and user registry;
* `events`: this option sets the collection of Windows event logs;
* `system`: this option sets the collection of system-related information;
* `network`: this option sets the collection of network data;
* `filesystem`: this option sets the collection of data related with NTFS and files;
* `malware`: this option sets the collection of system data that can be used to spot malware;
* `activity`: this option sets the collection of user activity data;
* `web`: this option sets the collection of browsing history and caches.

On the one hand, the usage of advanced tools set by the `sigcheck`, `density`, `iconsext` and `yara` options is independent of the configurations made to the collection of data types. On the other hand, directives under the respective main options of the data types are tied to them, meaning that they are disregarded if the main ones are set to `false`. For example, `memory-dump=true`, the option that instructs the tool to dump the Random Access Memory (RAM), is ignored if `memory=false`. The same goes for the `<option>-all` option, which sets all options of a certain data type to `true` for convenience, except `<option>-vss`. The script supports retrieving data from **all available VSS copies** by creating hard links to the copies via the Windows kernel namespace, a feature that can be turned on with `vss=true`. Each of the main options has its own `<option>-vss` option, which enables or disables the acquisition of VSS data for that particular data type. Note that the data collected by the `malware-startup` and `web-(chrome|ie|mozilla)` options is password-protected too, with the password being "infected" without quotes. All options not found or commented in the configuration file are set to `false` during runtime, including the password for the final compressed archive.

Further note that the `iconsext` option is useful to look for binaries compiled with unusual frameworks that set PE icons (*e.g.*, Python). Moreover, YARA rules need to have a `*.yar` file extension and to be put in the `tools-win\yara\rules\` folder. The output of all advanced tools are stored under the `malware` resulting folder.

Below is a minimal example of the configuration file setting the collection of the RAM, including the live and historical paged memory, the system registry and Windows event logs in text format, as well as the compression of the final data folder with password "infected" (without quotes). Note that this configuration skips the collection of historical system registry files.

```
# ir-rescue-win configuration file
# accepted values: 'true' or 'false' (exclusive)

# general
killself=false
sdelete=false
zip=true
zpassword=infected
ascii=false

# modules
memory=true
registry=true
events=true

vss=true

# memory 
memory-all=false
memory-vss=true
memory-dump=true
memory-pagefile=true

# registry
registry-all=false
registry-vss=false
registry-system=true

# events
events-all=false
events-txt=true
```

# Third-Party Tool List and References

## Windows

* **Sysinternals**: the [Sysinternals](https://technet.microsoft.com/en-us/sysinternals/ "Sysinternals Web Site") tools have been mostly developed by Mark Russinovich and are free to use under the [Sysinternals Software License Terms](https://technet.microsoft.com/en-us/sysinternals/bb469936.aspx "Sysinternals Software License Terms"). The full list of tools used by *ir-rescue-win* is `accesschk[64].exe` (v6.02), `autoruns[64].exe` (v13.62), `autorunsc[64].exe` (v13.61), `handle[64].exe` (v4.1), `Listdlls[64].exe` (v3.2), `logonsessions[64].exe` (v1.4), `ntfsinfo[64].exe` (v1.2), `psfile[64].exe` (v1.03), `PsGetsid[64].exe` (v1.45), `Psinfo[64].exe` (v1.78), `pslist[64].exe` (v1.4), `psloggedon[64].exe` (v1.35), `psloglist.exe` (v2.71), `PsService[64].exe` (v2.25), `sdelete(32|64).exe` (v2.0), `sigcheck[64].exe` (v2.52), and `tcpvcon.exe` (v3.01).

* **NirSoft**: the [NirSoft](http://www.nirsoft.net/ "NirSoft Web Site") suite of tools are developed by Nir Sofer and are released as freeware utilities. The full list of tools used by *ir-rescue-win* is `AlternateStreamView[64].exe` (v1.51), `BrowsingHistoryView[64].exe` (v1.86), `ChromeCacheView.exe` (v1.67), `DriverView[64].exe` (v1.47), `iconsext.exe` (v1.47), `IECacheView.exe` (v1.58), `LastActivityView.exe` (v1.16), `MozillaCacheView.exe` (v1.69), `nircmdc[64].exe` (v2.81), `OfficeIns[64].exe` (v1.20), `USBDeview[64].exe` (v2.61), and `WinPrefetchView[64].exe` (v1.35).

* **Cygwin**: the [Cygwin](http://www.cygwin.com/ "Cygwin Web Site") project is open-source and is used by *ir-rescue-win* only to filter outputs with the `tr.exe` (v8.24-3) and `grep.exe` (v2.21) utilities, using the 32-bit DLLs.

* **The Sleuth Kit (TSK)** (v4.3.0): the [TSK](http://www.sleuthkit.org/ "TSK Web Site") is an open-source forensic tool to analyze hard drives at the file system level, used by *ir-rescue-win* only to walk the MFT with `fls.exe`.

* **7za.exe** (v9.20): [7-Zip](http://www.7-zip.org/) is an open-source compression utility developed by Igor Pavlov and release under the GNU LGPL license.

* **winpmem_1.6.2** (v1.6.2): the [Pmem](https://github.com/google/rekall "Rekall GitHub Repository") suite is part of the open-source Recall memory analysis framework, used by *ir-rescue-win* to dump the memory.

* **md5deep[64].exe** (v4.4): the [md5deep](http://md5deep.sourceforge.net/ "md5deep Web Site") utility is open-source and is maintained by Jesse Kornblum.

* **LECmd.exe** (v0.9.2.0) and **JLECmd.exe** (v0.9.6.1): [LECmd](https://github.com/EricZimmerman/LECmd "LECmd GitHub Repository") and [JLECmd](https://github.com/EricZimmerman/JLECmd "JLECmd GitHub Repository") are open-source, MIT-licensed parsers for Link (LNK) and for automatic and custom destinations jump lists with support for Windows 7 thru Windows 10, respectively. These are developed by Eric Zimmerman.

* **rifiuti-vista[64].exe** (v.0.6.1): [Rifiuti2](https://github.com/abelcheung/rifiuti2 "Rifiuti2 GitHub Repository") is an open-source parser for the recycle bin released under the BSD license.

* **RawCopy[64].exe** (v1.0.0.15) and **ExtractUsnJrnl[64].exe** (v1.0.0.3): [RawCopy](https://github.com/jschicht/RawCopy "RawCopy GitHub Repository") (essentially, a combination of **ifind** and **icat** from TSK) and [ExtractUsnJrnl](https://github.com/jschicht/ExtractUsnJrnl "ExtractUsnJrnl GitHub Repository") are open-source NTFS utilities to extract data and special files developed by Joakim Schicht.

* **densityscout[64].exe** (build 45): the [DensityScout](https://www.cert.at/downloads/software/densityscout_en.html "DensityScout Web Site") utility to compute entropy was written by Christian Wojner and is released under the ISC license. 

* **YARA** (v3.5.0): [YARA](http://virustotal.github.io/yara/ "Yara Web Site") is an open-source signature scheme for malware that can be used to perform scans of specific indicators.

## Unix

* **linpmem-2.1.post4** (v2.1.post4): the [Pmem](https://github.com/google/rekall "Rekall GitHub Repository") suite is part of the open-source Recall memory analysis framework, used by *ir-rescue-nix* to dump the memory.

