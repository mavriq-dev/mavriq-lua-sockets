### Mavirq Lua Sockets

This are static builds of lua sockets for use with Reaper. They have lua staticlly linked in.

Receipe:

download latest [lua sockets](https://github.com/lunarmodules/luasocket)

make lua version you want ie. 5.3.6
Note:
on linux version you must:
```
	MYCFLAGS=fPIC
	MYLDFLAGS=fPIC
```

take liblua.a which is the static build and place in luasockets `/src` directory

makefile in luasockets `/src` needs 2 changes


line 166 needs to be changed `LDFLAGS_macosx= -llua -bundle -o #-bundle -undefined dynamic_lookup  -o `
line 277 needs to have lua version removed and be named core `SOCKET_SO=core.$SO  #socket-$(SOCKET_V).$(SO)`

linux needs
```
CFLAGS_linux=$(LUAINC:%=-I%) $(DEF) -Wall -Wshadow -Wextra \
	-Wimplicit -O2 -fPIC 
#LDFLAGS_linux=-O -shared -fpic -o
LDFLAGS_linux= -O -Wl,-Bstatic -L. -llua -Wl,-Bdynamic -shared -fPIC  -o
```
no point in code signing mac code. Unless it is signed by the same team as REAPER you will still get a security warning. Might as well leave unsigned. The code to sign is comment out in the github action.