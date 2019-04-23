# A imporved copy of `enum` by `Jordan Ritter <jpr5@darkridge.com>`

`enum` is tool to enumerate, using null and user sessions, Win32 (NT) information.

To build and run `enum` on Windows 10, several modifications needed, change `char*` like parameter data type to `wchar_t*`:

```
L381:
	ret = NetShareEnum((wchar_t*)wserver,level,(unsigned char **)&buff,prefmaxlen,
		               &read,&total,&resume);

L544:

	if (NetUserModalsGet((wchar_t *)&wserver,1,(unsigned char **)&buff) == NERR_Success) {
		PUSER_MODALS_INFO_1 info = (PUSER_MODALS_INFO_1)buff;

L761:

	ret = NetLocalGroupGetMembers((wchar_t*)&wserver, info[i].lgrpi0_name, 3, 
		                          (unsigned char **)&buff, 1024, &read, &total, &resume);
```
Remove errors:
```
enum.cpp(381): error C2664: 'DWORD NetShareEnum(LPWSTR,DWORD,LPBYTE *,DWORD,LPDWORD,LPDWORD,LPDWORD)': cannot convert argument 1 from 'char *' to 'LPWSTR'
enum.cpp(382): note: Types pointed to are unrelated; conversion requires reinterpret_cast, C-style cast or function-style cast
enum.cpp(506): warning C4477: 'printf' : format string '%d' requires an argument of type 'int', but variadic argument 1 has type 'LARGE_INTEGER'
enum.cpp(544): error C2664: 'DWORD NetUserModalsGet(LPCWSTR,DWORD,LPBYTE *)': cannot convert argument 1 from 'const unsigned short *' to 'LPCWSTR'
enum.cpp(544): note: Types pointed to are unrelated; conversion requires reinterpret_cast, C-style cast or function-style cast
enum.cpp(761): error C2664: 'DWORD NetLocalGroupGetMembers(LPCWSTR,LPCWSTR,DWORD,LPBYTE *,DWORD,LPDWORD,LPDWORD,PDWORD_PTR)': cannot convert argument 1 from 'const unsigned short *' to 'LPCWSTR'
enum.cpp(762): note: Types pointed to are unrelated; conversion requires reinterpret_cast, C-style cast or function-style cast

```

Require Visual Studio 2017 Community version installed.

Setup development environment with command:
```
> %comspec% /k "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\VsDevCmd.bat"

```

I use command line `cl` to build.
```
> cl -nologo /D_USE_32BIT_TIME_T  enum.cpp getopt.cpp Mpr.lib Netapi32.lib Advapi32.lib

```

Using preprocessor 
```
/D_USE_32BIT_TIME_T 
```
to remove error:
```
enum.cpp(228): error C2664: 'char *ctime(const time_t *const )': cannot convert argument 1 from 'const long *' to 'const time_t *const '
enum.cpp(228): note: Types pointed to are unrelated; conversion requires reinterpret_cast, C-style cast or function-style cast

```


## `enum` example

Using the user/password to login Windows in option -u user and -p password.
You can use localhost ip `127.0.0.1` for testing.

### `enum` Usage
```
enum.exe

usage:  enum.exe  [switches]  [hostname|ip]
  -U:  get userlist
  -M:  get machine list
  -N:  get namelist dump (different from -U|-M)
  -S:  get sharelist
  -P:  get password policy information
  -G:  get group and member list
  -L:  get LSA policy information
  -D:  dictionary crack, needs -u and -f
  -d:  be detailed, applies to -U and -S
  -c:  don't cancel sessions
  -u:  specify username to use (default "")
  -p:  specify password to use (default "")
  -f:  specify dictfile to use (wants -D)

```

### -U:  get userlist
```
> enum.exe -u xx -p xx -U 10.0.2.7
username: xx
password: xx
server: 10.0.2.7
setting up session... success.
getting user list (pass 1, index 0)... success, got 4.
  Administrator  Guest  xx  HomeGroupUser$
cleaning up... success.
```

### -G:  get group and member list
```
> enum.exe -u xx -p xx -G 10.0.2.7
username: xx
password: xx
server: 10.0.2.7
setting up session... success.
Group: Administrators
xx-PC\Administrator
xx-PC\xx
Group: Backup Operators
Group: Cryptographic Operators
Group: Distributed COM Users
Group: Event Log Readers
Group: Guests
xx-PC\Guest
Group: IIS_IUSRS
NT AUTHORITY\IUSR
Group: Network Configuration Operators
Group: Performance Log Users
Group: Performance Monitor Users
Group: Power Users
Group: Remote Desktop Users
Group: Replicator
Group: Users
NT AUTHORITY\INTERACTIVE
NT AUTHORITY\Authenticated Users
Group: HomeUsers
xx-PC\xx
xx-PC\Administrator
xx-PC\HomeGroupUser$
cleaning up... success.

```
### -N:  get namelist dump (different from -U|-M)
```
> enum.exe -u xx -p xx -N 10.0.2.7
username: xx
password: xx
server: 10.0.2.7
setting up session... success.
getting namelist (pass 1)... got 4, 0 left:
  Administrator  Guest  xx  HomeGroupUser$
cleaning up... success.
```

### -S:  get sharelist
```
> enum.exe -u xx -p xx -S 10.0.2.7
username: xx
password: xx
server: 10.0.2.7
setting up session... success.
enumerating shares (pass 1)... got 4 shares, 0 left:
  ADMIN$  C$  IPC$  Users
cleaning up... success.
```

### -P:  get password policy information
```
> enum.exe -u xx -p xx -P 10.0.2.7
username: xx
password: xx
server: 10.0.2.7
setting up session... success.
password policy:
  min length: none
  min age: none
  max age: 42 days
  lockout threshold: none
  lockout duration: 30 mins
  lockout reset: 30 mins
cleaning up... success.
```
### -M:  get machine list
```
> enum.exe -u xx -p xx -M 10.0.2.7
username: xx
password: xx
server: 10.0.2.7
setting up session... success.
getting machine list (pass 1, index 0)... success, got 0.
cleaning up... success.
```

