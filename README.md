# Fix_Dolphin_File_Associations
Fixing File Associations in Dolphin  ( Hyprland / Wayland / KDE6 )


🛠️ A practical guide to repairing broken file associations in the **Dolphin file manager** when running on **Hyprland** with **KDE Frameworks 6** under **Wayland**.

---

## 📋 Overview

Dolphin may not correctly open files in their associated applications when used under Hyprland + KDE6. This is due to missing or misconfigured components such as:

- Missing `applications.menu`
- Broken or user-defined MIME types (e.g. `text/plain`)
- Corrupt KDE service cache (`KSycoca`)
- Incorrect or outdated `.desktop` assignments

This guide walks you through the steps to fix it.

---

## ✅ System Requirements

- **Operating System**: Arch / Manjaro Linux  
- **Environment**: Hyprland + KDE Frameworks 6  
- **File Manager**: Dolphin  

---

## 🔧 Fix Steps

### 1. Update system MIME database

```bash
$ sudo update-mime-database /usr/share/mime
```

### 2. Clear KDE service cache (KSycoca)

```bash
$ rm ~/.cache/ksycoca6_*
```

Then **logout and log back in** to your KDE/Wayland session.

---

### 3. Fix missing `applications.menu`

Check if it exists:

```bash
$ ls /etc/xdg/menus/applications.menu
```

If it’s missing, create a symlink:

```bash
$ sudo ln -s /etc/xdg/menus/plasma-applications.menu /etc/xdg/menus/applications.menu
```

Then regenerate cache:

```bash
$ rm ~/.cache/ksycoca6_*
$ kbuildsycoca6 --noincremental --track menu
$ xdg-mime default org.kde.kate.desktop text/plain
```

Restart Dolphin:

```bash
$ killall dolphin
$ dolphin &
```

---

### 4. Verify file association

```bash
$ xdg-mime query default text/plain
# Should return:
org.kde.kate.desktop
```

---

### 5. Edit `mimeapps.list` manually

```bash
$ cd ~/.config
$ nano mimeapps.list
```

Replace invalid entries like:

```ini
text/plain=kate6;
```

With correct ones:

```ini
text/plain=org.kde.kate.desktop;
```

⚠️ Make sure each line ends with a `;`

---

## 🧪 Test

Open the following file types in Dolphin:

- `.txt`, `.jpg`, `.mp3`, `.mp4`, `.odt`, etc.

They should now open in their **default applications** automatically.

---

## 📝 Notes

- Run `kbuildsycoca6` every time you change menu or `.desktop` files.

---

## 🧩 Optional: Manually create `.desktop` or `applications.menu`

See full guide in [Fixing_Dolphin_and_Hyprland.md](Fixing_Dolphin_and_Hyprland.md).


