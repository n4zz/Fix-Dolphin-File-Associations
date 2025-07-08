-----------------------------------------------------------------------------  
# Guide: Fixing File Associations in Dolphin  


- **Warning:** ! Use at your own risk !  
- **Version:** 2025-07  
- **System:** Manjaro / Arch Linux + Hyprland + KDE Frameworks 6  

------------------------------------------------------------------------------  

## Wayland/KDE6/Hyprland and the Dolphin File Manager  

Dolphin works well under Hyprland, but it does not open files in associated applications by default.  

------------------------------------------------------------------------------  

## What does this mean, and what are the consequences?  
1. The system menu file (applications.menu) is missing  
   This file is part of the freedesktop.org standard, which KDE uses for categorizing applications. If it's missing, it can affect loading of file associations.  
       
2. The text/plain MIME type is "dirty" in user packages, but lacks a corresponding system definition  
   This might indicate the core MIME type text/plain is missing or broken in the system database (/usr/share/mime).  
       
3. The KSycoca database is broken or invalid  
   The error kf.service.services: unexpected object entry means there’s a problem in KDE’s service cache.  

------------------------------------------------------------------------------  

## Recommended Fix Steps  


### 1. Verify and update system MIME databases  

Regenerate the MIME database:  

    `sudo update-mime-database /usr/share/mime`  

This restores core system MIME type definitions.  


______________________________________________________________________________  
### 2. Clear and regenerate KDE cache (KSycoca)  

The KDE service cache might be broken:  

    `rm ~/.cache/ksycoca6_*`  

Then log out and back in to your KDE/Wayland session.  


______________________________________________________________________________
### 3. Check if /etc/xdg/menus/applications.menu exists  

    `ls /etc/xdg/menus/applications.menu`  

If missing applications.menu:  

    `ls -l /etc/xdg/menus`  

output:  

`plasma-applications.menu`  

*shows plasma-applications.menu (typical in KDE6 installations)*  


  * 3.1 Create a symlink  

        `sudo ln -s /etc/xdg/menus/plasma-applications.menu /etc/xdg/menus/applications.menu`


  * 3.2 Clear KDE cache again

        `rm ~/.cache/ksycoca6_*`  


  * 3.3 Regenerate the database  

        `kbuildsycoca6 --noincremental --track menu`  


  * 3.4 Fix MIME association  

        `xdg-mime default org.kde.kate.desktop text/plain`  


  * 3.5 Restart Dolphin (Or log out and back in)  

        `killall dolphin`  
        `dolphin &`  


  * 3.6 Fixing mimeapps.list  

        `cd/home/$USER/.config`  

        `sudo nano mimeapps.list`  


    **You will see something like this*:

        [Added Associations]  
        application/pdf=zathura.desktop;  
        application/x-python=pycharm.desktop;  
        audio/mpeg=qmmp.desktop;  
        image/jpeg=org.kde.gwenview.desktop;  
        text/html=code-oss.desktop;firefox.desktop;  
        text/plain==org.kde.kate.desktop;  

        [Default Applications]  
        application/x-extension-html=firefox.desktop;  
        application/x-extension-shtml=firefox.desktop;  
        application/x-extension-xht=firefox.desktop;  
        application/x-extension-xhtml=firefox.desktop;  
        application/xhtml+xml=firefox.desktop;  
        inode/directory=org.kde.dolphin.desktop;  
        text/calendar=org.mozilla.Thunderbird.desktop;  

    **Important:** Every line must end with a semicolon `(;)`.  


    Replace all entries with their appropriate .desktop file names.  
    For example:  
    change: *text/plain=kate-6.desktop;*  
    to: *text/plain=org.kde.kate.desktop;*  


  * 3.7 Verify using this command:  

        `xdg-mime query default text/plain`  
       output:  

       `org.kde.kate.desktop`  


______________________________________________________________________________
### 4. Test  

Try opening the following file types in Dolphin:  

    `• .txt, .jpg, .png, .mp3, .mp4, .odt, etc.`  

All files should open in their default applications without prompting.  


______________________________________________________________________________
### 5. Notes  

- Dolphin and kcmshell6 filetypes both read applications via applications.menu  
- You must run kbuildsycoca6 after any change to menu files or .desktop files  


______________________________________________________________________________
### 6. Alternative: Manual .desktop file for text/plain if above steps fail  

Create the file “.desktop“:  

    `sudo nano ~/.local/share/applications/org.kde.kate.desktop`  

Paste:  

    [Desktop Entry]  
    Name=Kate  
    Exec=kate %U  
    Icon=kate  
    Type=Application  
    Terminal=false  
    Categories=Utility;TextEditor;  
    MimeType=text/plain;  

Then run:  

    `update-desktop-database ~/.local/share/applications`  

______________________________________________________________________________
### 7. Alternative: Manually create applications.menu  

   * 7.1 Create the directory (if needed):  

      `sudo mkdir -p /etc/xdg/menus`  


   * 7.2 Create the file:  

      `sudo nano /etc/xdg/menus/applications.menu`  

   Paste:  

    <!DOCTYPE Menu PUBLIC "-//freedesktop//DTD Menu 1.0//EN"  
    "http://www.freedesktop.org/standards/menu-spec/1.0/menu.dtd">  
    <Menu>  
    <Name>Applications</Name>  
    <DefaultAppDirs/>  
    <DefaultDirectoryDirs/>  
    <Include>  
        <All/>  
    </Include>  
    </Menu>  
 
   This menu file tells KDE: “Use all applications from default locations (/usr/share/applications and ~/.local/share/applications) and create a full list.”  
   Save (Ctrl+O, Enter) and close (Ctrl+X).  

______________________________________________________________________________
### 8. Conclusion  

After completing this guide, Dolphin under Hyprland and KDE6 will be fully capable of opening files based on their type — no need to manually select applications each time.
