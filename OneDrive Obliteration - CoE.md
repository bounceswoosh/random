# How I obliterated my OneDrive data and got it back

TLDR: If you accidentally delete all of your files from OneDrive, your best bet is to restore it from the online OneDrive Recycling Bin*. Also, the Microsoft ecosystem has roughly 400 different ways to restore lost data, so the odds of retrieving your data are good. I panicked and did just about everything wrong, and in the wrong order, but I still got it all back in the end. Pretty much.

*Shared OneDrive folders need to be restored from the owner's online Bin or from your local Bin. Probably.


## Incident
On the afternoon of Sept 21, 2024, I noticed that my linux machine's logs were spewing disk space errors. I realized that I had inadvertently set up the [onedrive client](https://github.com/abraunegg/onedrive) for the wrong user, and it was attempting to sync the entire contents of my OneDrive account to my root partition, which was too small to hold it all. I deleted the entire OneDrive directory within that account to free up space.

Shortly thereafter, I discovered that everything I have saved to OneDrive account was missing, including the shared folders to which I had access.

Despite making just about every conceivable mistake in my panic, eventually I was able to recover all (probably?) of my files. This is my story.

## Sequence of Events

* Selected all files from Windows Recycle Bin and compressed them into an external file
* Discovered that the filenames in the compressed file were garbage
* Deleted the zipped file via File Explorer
* Because the zipped file was so big, the original files were deleted from the Recycle Bin and unrecoverable
  * I had considered deleting from the command line, but I'd just been bitten by aggressive deleting and was irrationally apprehensive. I totally forgot about shift-delete.
* Used OneDrive Premium's Restore to go back to "suggested recovery point" The suggestion was exactly the right change - but after I restored to that point, I saw that my OneNote directory was missing. It turns out that was a OneNote weirdness / red herring, but it led me to believe that the suggested recovery point was wrong.
* Checked `journalctl --user` on my linux box to identify the timestamp when deletions started
* Several iterations of using OneDrive's restore plus its online Recycle Bin. 
* I ended up using a week-old restore point and then restoring the items that the restore deleted from the online Recycle Bin. Spot checking seems good, but it's possible I missed some data. In theory I should have all the files from a week ago and all the files that were fully deleted after that, but will have a one week gap for files that merely changed. For my purposes, that is good enough. If I run across missing changes in files, I think I may be able to get them back because ... 
* I had an Excel file I'd modified in the previous week. I was able to restore the latest version from Excel's version history. I think that means that OneDrive restore writes over the previous version of the file, rather than performing a rollback.
* OneNote was a whole different and confusing situation. I had OneNote using a "local" directory that was also sync'd to OneDrive. Because my most recent file changes were in OneNote, I was looking for that directory to determine whether I'd recovered all my files. The files for my notebook never appeared in any of my restore iterations. This contributed to my early stage flailing.
* When for some reason I finally opened up the web version of OneNote, the most recent contents of my notebook were right there. I didn't lose anything, but none of it is coming from the place it was stored prior to this event. Online OneNote notebooks seems to exist in some mysterious pocket dimension, untouchable via the OneDrive interface. I still don't understand why my OneNote directory disappeared, unless I ran afoul of some behind-the-scenes scaffolding between OneDrive and OneNote.

## Another Approach
I'd also deleted all the files that my husband had shared with me. I think these may have been present in my local Recycle Bin, but they were not present in my online Recycle Bin.
* He tried using the context menu "Restore Previous versions" for one file from Windows, thinking it's tied to OneDrive. It's not; he had Windows Shadow Storage running and didn't know it.
* He then just ... restored everything in his Recycle Bin. There was a hiccup because of the previous version of that one file, but it all worked out and he avoided all the mess that I went through.

## Lessons Learned
* It's really hard to think straight in a situation like this.
* If I accidentally delete data from OneDrive, **even a lot of data**, I should simply use the online or local recycling bin to restore it.
* I already knew RAID is not backup. This is a reminder that Sync/Mirroring is not Backup - although OneDrive's Recycling Bin and restore points significantly reduce the risk of permanent loss. 
  * Caveat: Restore points are only available in the Premium version.

### OneDrive Product / Service
* The premium version of OneDrive can do a checkpoint restore with certain predetermined intervals. You can also restore to an exact change if it's within two days.
* Unlike the Windows Recycle Bin, the online OneDrive Recycle Bin has no size limit.
* OneNote is voodoo.

### OSS onedrive client for linux
* [abraunegg's onedrive client](https://github.com/abraunegg/onedrive/), at least in the debian package, runs as a per-user systemctl "unit."
* Running the `onedrive` command creates the default configuration in `~/.config/onedrive`
* The default configuration is to sync every file bi-directionally.
* The default configuration requests a read/write refresh token. You can instead configure it to use a read-only token.
* The onedrive client behaves like OneDrive does on other platforms, like Windows and Mac - bi-directionally. I wouldn't have just deleted everything from my OneDrive-backed directories on my Windows machine, so I'm not sure why I thought that it would be any different on linux. Panic about disk space, I guess.
  * I don't know what the client does if you run out of disk space and DON'T delete any files.

### systemctl
* `systemctl --user` exists and is similar to `systemctl`, but runs for an individual user. It exists entirely in parallel to the main `systemctl` and does not interact with it.
  * https://wiki.archlinux.org/title/Systemd/User
* Because services configured under `systemctl --user` are for, well, users - if you make changes as the root user, you will in fact set it up for the root user. (Note to self: `su -` was okay 25+ years ago when I started using linux, but I need to start using `sudo` even on home machines. If I had tried `sudo systemctl --user` instead of being logged in as root, it would have failed and I wouldn't have been able to accidentally set up a sync for the root user.)


### Windows 11
* You can restore files from the Windows Recycling Bin (duh), but you could *also* choose to copy the files to other directories instead of restoring them to their original location.
* Don't create a compressed file (e.g. zip) from the Windows Recycling Bin. The top level (whether directory or file) in the archive will be renamed to gibberish. But the zip file WILL have all the data; it just isn't reasonable to use it for a bulk restore.
* Windows does not prompt you for a hard delete when you delete a large file (use Shift-Delete to hard delete, or use the command line).
* If you delete a large file and it goes to the local Recycle Bin, Windows will silently purge older files in the Recycle Bin.
* Windows 11 supports file versioning in at least two ways.   
  * **System Restore**: If you enable this via `System Properties -> System Protection` or `Create a Restore Point`, the file context menu "Restore previous versions" will show you previous snapshots of the file. It appears that the context menu is always visible, even if you don't have it enabled. This uses local data and does NOT reflect the versioning in OneDrive. You can see if it's enabled via `vssadmin list shadowstorage`. I now have this enabled.
  * **File History**: This is enabled via `Control Panel -> File History` and allows you to save copies of your files to an external drive or network share. I haven't tried this.
* There's also `Windows Backup`, which I think is a superset of OneDrive, but I'm not sure. 

### OneNote
* OneNote notebook storage is still a mystery to me. There's a note about this [here](https://github.com/abraunegg/onedrive/blob/master/docs/usage.md), but that doesn't really explain anything. I guess it's sort of like how Google Photos and Google Drive are completely separate, but both contribute to your overall storage usage.
* OneNote stores backups of each section locally under `%USERPROFILE%\AppData\<thingie>\Microsoft\OneNote\16.0\Backup`. You can configure it to snapshot as often as you'd like and keep as many copies as you'd like.
* Big caveat - the backups are stored as individual sections. To use them you'd need to import them each into a notebook and recreate the previous structure.
* You can export the entire notebook as a `onepkg` file for backups. I don't think there's a handy way to automate this.