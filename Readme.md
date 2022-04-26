[![License](https://img.shields.io/badge/license-GPLv3-orange)](./LICENSE)
![Relase Build](https://img.shields.io/github/workflow/status/mavriq-dev/mavriq-lua-sockets/Build?label=Build)
![Code Size](https://img.shields.io/github/languages/code-size/mavriq-dev/mavriq-lua-sockets)
![Repo Size](https://img.shields.io/github/repo-size/mavriq-dev/mavriq-lua-sockets)
![Release Ver](https://img.shields.io/github/v/release/mavriq-dev/mavriq-lua-sockets)
![Prerelase Ver](https://img.shields.io/github/v/release/mavriq-dev/mavriq-lua-sockets?include_prereleases)
# Mavirq Lua Sockets

### Background
Reaper is missing Lua Auxlib in it's embedded version of Lua. As such things such as the luasockets library etc will not work. If loaded they will throw an error for missing symbols when the library tries to access those in the missing AuxLib.

Until the REAPER devs fix this, we have to work around the issue. This project does that for all all three REAPER platforms.

So far I have built the Lua ZeroBrane Debugging based on this package. Mavriq LuaSockets comes with support for raw, UDP, SMTP, FTP and HTTP. Sorry HTTPS isn't available yet, but there is a sister package I will soon incorporate.

Danial Lumertz has an excellent thread [here](https://forums.cockos.com/showthread.php?t=265870) with example how to use it.

### Note to REAPER Devs

**This may be a one line fix depending on how Lua was added to Reaper.** Typically Lua is embedded by including the headers like the code below. If so the header for `lauxlib.h` is missing.

<pre>extern "C"
{
    #include "lualib/lua.h"    
    <b>#include "lualib/lauxlib.h"</b>
    #include "lualib/lualib.h" 

}</pre>

### The Solution

To fix the missing library we need to build a version of sockets with the Lua library statically linked. This will "override" the version of Lua built into reaper. All Lua calls by the sockets library will use the statically linked version.

### Binaries
You can find builds under releases on GitHub, or you can install via [ReaPack](https://github.com/mavriq-dev/public-reascripts/raw/master/index.xml).

### Secret Sauce:
Each platform has it's own quirks in making a library. In addition I wanted to come up with a solution that could be built entirely from the command line so it could be automated. You can find the workflow YAML under the `Actions` tab in `GitHub` for the project. These are NOT step by step instructions but a how to for the general concept.

| Platform  | Description  |
|---|---|
| Windows |  `Lua Library`</br>Download the official 5.3.6 static binaries from [SourceForge](https://sourceforge.net/projects/luabinaries/files/5.3.6/Windows%Libraries/Static/lua-5.3.6_Win64_vc16_lib.zip).</br></br>`Luasockets`</br>Clone the git repository for luasocket [https://github.com/lunarmodules/luasocket](https://github.com/lunarmodules/luasocket). Some changes are needed to the Visual Studio solution included with the project, or you can use the MSBuild command line tool:<pre>msbuild socket.vcxproj </br>/p:Configuration=Release /p:Platform=x64 /property:AdditionalDependencies=lua53.lib </br>/property:WindowsTargetPlatformVersion=10.0.19041.0</pre> |
| macos | `Lua Library`</br>Download the official 5.3.6 sources from [https://www.lua.org/ftp/lua-5.3.6.tar.gz](https://www.lua.org/ftp/lua-5.3.6.tar.gz) and build.</br></br>`Luasockets`</br>Clone the git repository for luasocket [https://github.com/lunarmodules/luasocket](https://github.com/lunarmodules/luasocket). Several changes are needed in the makefile under the src directory:</br><PRE>LUAV?=5.3</br>LDFLAGS_macosx= -L. -llua -bundle -o</br>SOCKET_SO=core.$(SO)</PRE>Copy the static lua lib and needed headers to the src dir and build.</br></br> One issue with the mac version is code signing. A security message will pop up on the first load, and the user will need to add the appropriate permissions under the system settings. While it is possible to sign it, it makes little sense to do so. Modules have to be signed by the same team as the main app in macos. Since that isn't possible in this case, signing only changes the security message to something more confusing than the original message. There is code to sign the library in the workflow yaml that has been commented out if you want to do it anyway. |
| Linux | `Lua Library`</br>Download the official 5.3.6 sources from [https://www.lua.org/ftp/lua-5.3.6.tar.gz](https://www.lua.org/ftp/lua-5.3.6.tar.gz).</br></br>You need to change a couple of lines in the makefile in the src directory:</br><pre>MYCFLAGS= -fPIC</br>MYLDFLAGS= -fPIC</pre>or the library won't work when linked into luasockets.</br></br>`Luasockets`</br>Clone the git repository for luasocket [https://github.com/lunarmodules/luasocket](https://github.com/lunarmodules/luasocket). Several changes are needed in the makefile under the src directory:</br></br><PRE>LUAV?=5.3</br>LDFLAGS_linux=-O -Wl,-Bstatic -L. -llua -Wl,-Bdynamic -shared -fPIC -o</br>SOCKET_SO=core.$(SO)</PRE>Copy the static lua lib and needed headers to the src dir and build.|

