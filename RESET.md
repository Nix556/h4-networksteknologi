# How to restore old configs

copy running-config startup-config
configure replace nvram:startup-config force
