> blog https://www.maketecheasier.com/get-rid-screen-tearing-linux/
> 
Because Intel uses open source drivers, the Xorg configuration 
is going to be your most direct route. Create a file at 
`/etc/X11/xorg.conf.d/20-intel.conf`

```
Section "Device"
 
    Identifier "Intel Graphics"
 
    Driver "intel"
 
    Option "TearFree" "true"
 
EndSection
```