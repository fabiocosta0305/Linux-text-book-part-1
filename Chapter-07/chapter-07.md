# Introduction to Linux Editors, Shell Scripts, and User Profiles

![*Real programmers...*](images/Chapter-Header/Chapter-07/real_programmers-2.png "vi")

## Objectives

In this chapter we will be continuing our exploration of the Linux Shell.  We will be introducing editors and examining their use in managing our Linux system.  We will also look at understanding user environments and write our initial shell scripts.

* Understand the difference between stream editors and text editors
* Understand and learn how to use the vi(m) editor
* Understand how to use shell scripts to automate tasks
* Understand how to use the system PATH and modify a users profile

## Outcomes

At the outcome of this chapter a user will be able to use the vi editor for creating and manipulating text files and shell scripts.  This will give you mastery over the data on your system.  You will be comfortable creating a shell script to automate system administration tasks and you will understand how a user's system PATH and profile is modified and loaded.  This will open up the remaining chapters where we introduce additional complexity in writing shell scripts and enable greater system administration via shell scripts and using the vi editor.

## History of Unix/Linux Editors

In the previous chapter we continued learning about essential and additional command binaries that can be executed in the shell.  We learned about meta-characters which help expand our abilities to execute repetitive tasks and to simplify searching and file creation.  The next concept to introduce is __shell scripts__.  Shell scripts are a combination of all of the above features that can be placed into a single text file and run on demand or as a scheduled task.  This allows you to have prepared __shell scripts__ that can be copied from system to system allowing administrators to build up a *tool belt* of scripts that help them get common tasks done.  With that in mind, we come to the concept of the __editor__. In Unix/Linux due to the history and different types of standard in and standard out, there are also different types of editors.  The two categories are __stream editors__ and __screen editors__.  Overtime the distinction has blurred but it is safe to think in these two different categories mostly because each group retains artifacts from its development history.

### Stream Editors

The reason we call them __stream editors__ is that at their time of development, the modern day 30 inch screen we have, and even X for that matter, didn't exist.  So editing was done not via text editors or word processors, but it was done by editing single lines at a time.  One line would follow another line - hence a *stream*.  The two main editors represented in this category are __emacs__ and __vi__.  Each of these has evolved a distinct following.  The __vi editor__ (pronounced *vee-eye*) currently is a Unix Posix standard and the only editor that you will find installed by default on all Unix and Linux distros guaranteed, though some newer Linux distros are changing **vi** out for **nano**.  So learning __vi__ gives you the ability create shell scripts on any Unix/Linux system.

### Emacs

Emacs was originated in 1976 from the AI Labs at MIT, the same place Richard Stallman came from.  GNU Emacs was released in 1984 and developed entirely by Richard Stallman himself.  In 1980 James Gosling (father of the Java Language) had created his own Emacs in the spirit of opensource called gmacs, but sold his project to a company who re-licensed it with a proprietary license.  Emacs is basically a [Lisp language](https://en.wikipedia.org/wiki/Lisp_\(programming_language\) "Lisp")  interpreter focusing on macros (or key combinations) to make repeatable actions.  Emacs is a very powerful editor (see the cartoon at the beginning of the chapter) and has plugins for email and other functions to exist entirely inside of emacs.  In the course of this book we will not be focusing on emacs but that is not because of any deficiency, I recommend you to try it out at least once.

> *In it normal editing mode, GNU Emacs behaves like other text editors and allows the user to insert characters with the corresponding keys and to move the editing point with the arrow keys. Escape key sequences or pressing the control key and/or the meta key, alt key or super keys in conjunction with a regular key produces modified keystrokes that invoke functions from the Emacs Lisp environment. Commands such as save-buffer and save-buffers-kill-emacs combine multiple modified keystrokes [^80]*.
> *GNU Emacs is an extensible, customizable text editor—and more. At its core is an interpreter for Emacs Lisp, a dialect of the Lisp programming language with extensions to support text editing [^78] [^79].*

### The vi Editor

The other stream editor is the __vi editor__ or just __vi__ (pronounced *vee-eye*).  The creator of the __vi editor__ was [Bill Joy](https://en.wikipedia.org/wiki/Bill_Joy "Bill Joy") at UC Berkeley.  His intent was to extend the original ideas behind Ken Thompson's editor which was named *ed*.  The __vi__ editor is written in the C language but you don't code it in C, Unlike emacs which exposes it's LISP interpreter to the user.  The history of __vi__ varies widely from that of Emacs because __vi__ is not a GNU project.  The chart below shows the history of the __vi editor__.

  Editor           Year Released              Originator
------------     ------------------------   -------------------------------------
   ed                  1971                   Ken Thompson (original Unix)
   em                  1976                   [George Coulouris](https://en.wikipedia.org/wiki/George_Coulouris_\(computer_scientist\) "George Coulouris")
   ex                  1978                   Bill Joy, Charles Haley, included in BSD
   vi                  1979                   Bill Joy
   vim                 1991                   [Bram Moolenaar](https://en.wikipedia.org/wiki/Bram_Moolenaar "Vim")

Initially Ken Thompson's editor worked well for what he needed.  But the commands were very cryptic and made using the Thompson editor very difficult.  Ken Thompson's original shell *ed* is still available for [download on Ubuntu and Fedora](http://linuxclues.blogspot.com/2012/09/ed-tutorial-line-editor-unix.html "ed") if you are interested to see what it was like to use Unix back in the early 70's

Sometime in 1976 an AT&T co-worker, George Coulouris, while working on sabbatical at Queen Mary's College London, extended Thompson's editor and added some usability features.  Continuing the clever hack, he named the editor __em__ meaning *ed for mortals*. Editors at this time were designed for display terminals that did not have dedicated ram (expensive at the time).  The editors only modified a line at a time and were called [Line Editors](https://en.wikipedia.org/wiki/Line_editor "Line Editor").  Because of slow screens and the high price of memory you had the choice of using line editors or displaying the content of a file. Not until the 1980's did the concept of visual editing really catch on as technology made it possible.

In 1978/89, out at Berkeley, Bill Joy came into the picture.  He helped design an improved __em__ called __ex__, *em extended*.  This introduced a new visual mode in edition to the line editor features that everyone was used to.  This extension to __ex__ was called __visual mode__ or __vi__.  After one year and the changes in technology __ex__ shifted from being a *line editor* to a *visual editor* primarily.  Hence in 1979 by the time of the second BSD Unix release, __ex__ was hard linked to permanently launch in __vi__ mode. Thus, vi is not really the evolution of ex, vi is ex [^82].

### Relationship of vi and vim

In January of 1983 AT&T's UNIX System V adopted __vi__ as their standard editor.  This put __vi__ in the hands of everyone using commercial Unix from AT&T as well as anyone using BSD Unix--which up to that point meant almost everyone in the commercial world.  But it was not until June of 1987 that [Stevie](https://en.wikipedia.org/wiki/Stevie_\(text_editor\) "STEVIE") (ST editor for VI enthusiasts), a limited vi clone appeared. In early January, 1990, Steve Kirkendall posted a new clone of vi, Elvis, to the Usenet newsgroup comp.os.minix, aiming for a more complete and more faithful clone of vi than Stevie. It quickly attracted considerable interest in a number of enthusiast communities[^83]. Andrew Tanenbaum quickly asked the community to decide one of these two editors to be the vi clone in Minix;[^84] Elvis was chosen, and remains the vi clone for Minix today.

But at UC Berkeley, Keith Bostic wanted a "bug for bug compatible" replacement for Joy's vi for BSD 4.4 Lite. The original __vi__ code was encumbered by AT&T licensing because Bill Joy had extended Thompson's original code which technically belonged to AT&T. Using Kirkendall's Elvis (version 1.8) as a starting point, Bostic created [nvi](https://en.wikipedia.org/wiki/Nvi "nvi"), releasing it in Spring of 1994.[^85] FreeBSD and NetBSD continue to use nvi to this day.

In 1991, Bram Moolenaar created a port of __vi__ called __vim__ *vi improved*. Vim was created under a GPL compatible free license and it is compatible with the large majority of __vi__ functionality and extends to add some modern features like unlimited undo/redo for example.  Because of this __vim__ is available for all Linux based systems, BSD, Windows, and others.  Some distros link __vi__ to __vim__ replacing it out right.  

### vi has a Sharp Learning Curve

Many people will say that the __vi editor__ has a sharp learning curve and not to use it.  I believe that is a spurious argument.  The __vi editor__ is not a text editor comparable to notepad, but vi was a specific tool developed to create complex interaction with text in as few key strokes as possible. Learning to play the guitar is difficult in the beginning but once your have the muscle memory to do it you can become an expert player that can make beautiful music that few others can.  The power of __vi editor__ is in the ability to do line editing and visual editing all from the __vi editor__, the ability to search and find, execute internal commands, even use grep and regex for complex pattern matching and replacement from within vi.  Keeping your fingers on the keyboard constantly moving keeps your fingers and mind occupied. Nothing takes more time than to change "contexts". Don't abandon it because it is hard!  You will eventually be working on systems that have no GUI at all: FreeBSD or Ubuntu Server or RHEL or CentOS you will have to use vi.

#### vi and vim

By using vim as a text editor we can create shell scripts which are collections of shell commands with meta-characters and and some control logic. Ubuntu links to vim directly as seen in the image below.   Fedora keeps two distinct binaries __vi__ and __vim__ but both of then link back to __vim__.  You can use the `which vi` and/or `which vim` to find out the default state of vim on your distro.

### The 3 vi Modes

> __Example Usage:__ Let's invoke vim from the shell.  Open up a terminal and type ```vim notes.txt```, what happens?  You see a screen like this.  All those tilde marks (\~) mean that those lines don't exist--they are visual place holders. If you receive an error that `vim` is not installed you will need to install it via your package manager.

![*vi initial screen*](images/Chapter-07/editors/vi/vi-blank.png "vi blank")

The __vi editor__ has 3 modes:

1) COMMAND mode used to position the cursor
2) INSERT mode used to insert/delete text
3) EX mode used to issue commands that edit lines and change the display of the vi editor.

To transition from command mode to insert mode you use the __ESC__ key.  Hitting escape plus one of the text modification commands will automatically take you to __INSERT__ mode.  You will know you are in INSERT mode because the bottom of the screen will say INSERT.

> __Example usage:__  Let's type a hello world message in the vi editor. Continuing from the example above, to be able to insert text to the file you need to switch modes to INSERT mode.  Hit `ESC` then `i` and then type ```hello world!```  

> __Example usage:__ What happens when you hit an arrow key after typing ```hello world!```?  Why is this? Remember we need to switch modes between COMMAND mode and INSERT command.

## vi Command Cheat Sheet

There are over 150 distinct commands in vi.   But to be proficient you need to memorize *initially* about ~25 key commands.  I have provided those in the charts below.  Some of the commands automatically trigger INSERT mode after you execute them.  For instance the ```ESC then a``` command will append or add text after the end of the current line.  It makes sense that you would want to enter INSERT mode after typing an append command.  Remember to see the true advantage try to keep your fingers on the *home row* of the keyboard and case matters!

: Positional Commands That Trigger Insert Mode

  Command                 Command Description
-----------   ---------------------------------------------------
    a             add(append) text to the right of the cursor
    A             add(append) text to the left of the cursor
    i             insert text to the left of the cursor
    I           insert text at the beginning of the line of text
    o             add a new blank line below the cursor
    O             add a new blank line above the cursor
-----------   ---------------------------------------------------

: Line Position Commands That Trigger Insert Mode

  Command                 Command Description
-----------   ----------------------------------------------------------
w               moves the cursor to the beginning of the next word
W               same as above but space delimited
b               moves the cursor to the beginning of the previous word
B               same as above but space delimited
0               moves the cursor to the beginning of the current line
$               moves the cursor to the end of the current line
H               moves the cursor to the upper left corner
M               moves the cursor to the middle left position
L               moves the cursor to the lower left position
-----------   ----------------------------------------------------------

: Text Modification Commands That Do Not Trigger Insert Mode

  Command                 Command Description
-----------   ----------------------------------------------------------------
x                delete character
X                delete character before the cursor position
dd               deletes the line of the current cursor position
yy               yanks or copies the current line to the clipboard
p                pastes the current line or lines that are in the clipboard
\.                 repeats the previous command executed
 u                      undo previous command (unlimited)
 Ctrl + r               redo previous command (unlimited)
 Ctrl + g                       file info
 G                           go to last line of file
 ZZ              shortcut to save the current file and quit out of vi
-----------   ----------------------------------------------------------------

> __Example usage:__  Remember the previous example where we inserted text?  Now let's insert a new line of text.  How would we do it based the tables above?   We need to switch from INSERT mode back to COMMAND mode.  This time we type ```ESC then o``` to insert a newline below our cursor.  

![*vi newline insert*](images/Chapter-07/editors/vi/vi-shift-o.png "vi newline insert")

> __Example usage:__ What is the command sequence to delete a single character? You would switch to COMMAND mode by typing ```ESC then x```.

> __Example usage:__ What is the command sequence to delete an entire line?  You would switch to COMMAND mode by typing ```ESC then dd```.

> __Example usage:__ What command sequence would you use to move the cursor position to the end of the current line? You would switch to COMMAND mode by typing ```ESC then '$'```

### vi/ex Mode

  In addition to COMMAND mode and INSERT mode there is also one other mode.  This is called __EX__ mode.  This mode is the original __EX__ editor.  By hitting the ```ESC``` key and then the colon ```:``` you will now have the ability to enter additional commands not directly available in the other modes.  There are a series of commands you can execute while in __EX__ mode.  The most important is how to save and how to quit.

![*vi ex mode*](images/Chapter-07/editors/vi/vi-ex.png "vi in ex mode")  

: Save and Quit Commands

  Command                 Command Description
-----------   ----------------------------------------------------------------
 :w                  write the contents of a file (save)
 :q!                 force quit out of the vi editor ignoring any changes
 :wq                 write and quit a file in vi
 :wq!                force write and quit a file in vi
-----------   ----------------------------------------------------------------

: Additional Meta-Commands

  Command                 Command Description
-----------   --------------------------------------------------
 :set nu               make line numbers appear in vi
 :set nonu             turn off line numbering in vi
 :5               move the cursor to a specific line number
 :$               move the cursor to the end of the file
-----------   --------------------------------------------------

#### Search Forward

__EX__ mode also contains the ability to search for occurrences of text patterns within a text file while using __vi__.  By typing the combo ```ESC then /``` you see a slash at the bottom of line of the __vi__ editor. This allows you to search for a pattern from your current cursor position.  You can type ```ESC then ?``` to search the same pattern going up.   Hitting the letter *n* will move you to the next result which could be multiple lines away in a large document.  You can also use shell-metacharacters to search.  The next examples can be recreated using the log file located in the Chapter-07 directory of the included code under the *files* directory.  This example uses the file named ```u_ex150911.log``` which is an IIS webserver log for a WordPress Installation. Each of these commands needs to be prefaced by an ESC push to change modes.

\newpage

![*vi search*](images/Chapter-07/editors/vi/vi-search.png "vi search")

`/[Mm]ozilla`

: will search forwards for any lines containing either *Mozilla* or *mozilla* and highlight each occurrence in the file you are editing in __vi__.

#### Search Backwards

`?Mozilla`

: will search the file backwards for the word *Mozilla*.

`/Mozilla\/?\.0`

:  This is where we can combine shell meta-characters inside of __vi__ for searching for Mozilla versions

`?MSIE\\+[6-8]*`

:  This allows for backwards shell meta-character search. In this case notice the introduction of the escape character __\\__. Normally the __+__ sign has meaning but in our pattern we want to find all the old versions of Internet Explorer 6-8 that are visiting our blog.  To do this we pass the line escaping the __+__ because we want it to match as a text character not as a shell meta-character.

### vi/ex Mode Find and Replace Globally

__vi__ also has the ability to find and replace via a single line or globally.  By typing the ```ESC then :``` you will enter the same __ex__ mode mentioned above when learning about saving and quitting files.  See the sample file ```2016-05-31-using-s3cmd.md``` located in `files` > `Chapter-07` > `markdown` directory of the files folder.

`:s/Ubuntu/Fedora`

:  The *s* tells us it is a single find and replace or substitute.  This is a single instance replacement.

`:s/fall2020/spring2021/g`

:  This command the *s* tells us to substitute the word *fall2020* for the word *spring2021* and the trailing *g* means every occurrence on that line.

`:1,$s/&#47;/\//g`

: There is a wildcard option for the line range as well using the `%`

`:%s/&#47;/\//g`

:  This command has a range prefix, the *1* tells the replacement to start from line 1 and continue to line *$* which is the last line of the file, and replace all occurrences (replace all) of ```&#47;``` which is the html code for a ```/``` and note the escapes needed to replace it with a ```/```

`:47,86s/<br \/>//g`

:  This command tells us to do the replacement of lines 47-86 and strip out all the extranious ```<br />``` tags.  Note the backslash to escape the forward slash.

### Why vi Key Bindings are as They Are

When looking at the patterns of the key-bindings in __vi__ they seem a little strange.  The reason they were created the way they were had to due with the brand of terminal that __vi__ was created on.  Remember the standard IBM keyboard we are used to using wasn't created until 1981 on the [IBM PC 5150](https://en.wikipedia.org/wiki/IBM_Personal_Computer "5150"). The type of terminal and keyboard in use at UC Berkeley by Bill Joy was, at that time a competitor to the DEC VT 100 terminals, called the [ADM-3A terminal](https://en.wikipedia.org/wiki/ADM-3A "ADM-3A") [^81]. It happened that the ESC key was where the modern caps lock key is and that is why ESC is the key used to change modes. The convention just stuck, Unix is more about tradition than reason one could say.

![*Original ADM-3A Keyboard Layout*](images/Chapter-07/systems/640px-KB_Terminal_ADM3A-svg.png "ADM-3A layout")

#### A Note About Bill Joy

![*Bill Joy - creator of vi*](images/Chapter-07/people/384px-Bill_joy-2.png "Bill Joy")

In some ways Bill Joy could be seen as the west coast version of Ken Thompson.  Before Stallman left MIT and started GNU Bill Joy was working hard as a graduate student at Berkeley out in California.  He played a large part in helping to further develop BSD Unix.  Last chapter we mentioned that he created the C shell and he is also the creator of the vi editor.   He left Berkeley in 1982 with 3 other grads from Stanford to form Sun Microsystems, which would play a large role in the commercial Unix world and innovated many technologies (Java). Joy stayed on at Sun until 2003.

In the year 2000 Bill Joy wrote a seminal paper called, "[The Future Doesn't Need Us](http://archive.wired.com/wired/archive/8.04/joy_pr.html "The Future Doesn't Need Us")".  In the paper he was shocked by the speed of the progress scientific futurist community lead by [Ray Kurzweil](https://en.wikipedia.org/wiki/Ray_Kurzweil "Kurzweil")  coupled with how little they were examining ethics in the face of technological challenges. Ironically Ray Kurzweil is currently employed by Google, whose corporate motto until recently was [*Don't be evil*](https://en.wikipedia.org/wiki/Don%27t_be_evil "Google Motto").  Bill Joy said in quote,

> *"From the moment I became involved in the creation of new technologies, their ethical dimensions have concerned me, but it was only in the autumn of 1998 that I became anxiously aware of how great are the dangers facing us in the 21st century. I can date the onset of my unease to the day I met Ray Kurzweil, the deservedly famous inventor of the first reading machine for the blind and many other amazing things."*  

### Screen Editors

The second family of editors differs from the first in that they were created after X was fully implemented and are fully screen oriented, menu driven and have no concepts of what vi and emacs do in the way of line editor functions.  There is a second sub-category of screen editors called GUI editors.

#### GNU Nano

[GNU Nano](https://en.wikipedia.org/wiki/GNU_nano "Nano") was created in 2000 as a GPL replacement for a common non-free text editor that had come from the University of Washington called PINE.  The design in simpler than vim and has become a popular alternative.  Nano relies on using the *Control* key in combination with other keys for action.  For example ```^O``` to save and ```^X``` to quit a file--these commands are listed at the bottom of the screen. Unlike vim, there are no modes, so you are always in *insert* mode and can use the arrow keys and type as if you were in a regular GUI based text editor. For Fedora 33, GNU Nano is to replace vim as the default editor.  Nano is very similar to editors such as notepad but has features similar to vim and VSCode.  Nano is entirely rendered in text.

![*GNU Nano*](./images/Chapter-07/editors/nano/nano.png "GNU Nano Image")

GNU Nano was derived from two UNIX utilities originally used to edit/read/send email: PINE and PICO.  These were designed before GNU and Linux and did not have Free Software Licenses.  In 2001, GNU Nano was developed to give a free mail reader to the world.  The function of Nano changed as GUIs developed and became an regular text or file editor.

### GUI Text Editors

#### gedit

The [gedit](https://wiki.gnome.org/Apps/Gedit "gedit") program was released in 1999 -shortly before the GNOME desktop was released.  It is a full fledged text editor with plugin support and syntax highlighting.  It is currently part of the GNOME core applications and you will find it installed anywhere GNOME3 is installed.

#### Visual Studio Code

[Visual Studio Code](https://code.visualstudio.com/Docs/?dv=linux64 "Visual Studio Code") is a new comer to this field.  It is a text editor that has built in support for Git, plugins, and syntax highlighting. The advantage is that you can download VS Code for Windows, Mac, and Linux as the project is opensource.

#### Atom

[Atom](https://atom.io/ "atom") is a hackable text editor for the 21st century, built on Electron, and based on everything we love about our favorite editors. We designed it to be deeply customizable, but still approachable using the default configuration.

#### Leafpad and Mousepad

* [Leafpad](http://tarot.freeshell.org/leafpad/ "Leafpad") is an opensource notepad clone released in 2004.  Leafpad focuses on being light and having minimal dependencies. It provides syntax highlighting and is the default editor for [LXQT](https://lxqt.org/ "Leadpad default lxqt text editor") and was for Xfce until its replacement by Mousepad. Leafpad is built using GTK+.
* [Mousepad](https://en.wikipedia.org/wiki/Xfce#Mousepad "Mousepad") comes standard in Xfce as a notepad clone.  It is built using GTK3+.

#### Sublime

[Sublime](http://www.sublimetext.com/ "Sublime") is the missing editor for the Mac platform.  It is free to download but requires a license to be purchased.  It is a great product and if you will be doing any significant coding on the Mac platform this is really your only choice.  It is available on Windows and Linux as well.

#### Kate

[Kate](http://kate-editor.org/about-kate/ "Kate KDE") is the standard editor for KDE based desktops.  The KDE equivalent of gedit but more than gedit--Kate can be used as components of larger applications not unlike Emacs.  You can install this on any Linux based distro but it requires that KDE components be installed too.  If you are using Fedora or Ubuntu this makes the download and install much larger and may add files you don't necessarily want to your system.  If you desire to use Kate you may want to look at installing Kubuntu-desktop or finding a KDE based Fedora spin.

## Creating Shell Scripts

Let's open up a terminal and create a shell script.  To do this type: ```vi list-ip.sh``` and from here you will see the screenshot we saw earlier--blank.   The first thing we need to type is a shell directive or more commonly called, **she-bang**.  Although we are using the *bash* shell you can create scripts that can be run with other shells.  The first line of a shell script overrides the default shell and runs the script with the shell you determine.

The first line of any bash script should include:  ```#!/bin/bash``` to make sure that our script is executed with the bash shell.  Normally the ```#``` means a comment, but with the ```!``` after it followed by a path, the comment function is overruled.   ```#!``` can also be pronounced *crunch* *bang*.  

Not all systems store the bash binary in ```/bin/bash```.  You will need to check before you hard code that value.  You can use the ```which``` command that will show you the paths to the binaries.  

![*which bash on Ubuntu*](images/Chapter-07/editors/bash/which-ubuntu.png "which Ubuntu")  

![*which bash on Fedora*](images/Chapter-07/editors/bash/which-fedora.png "which Fedora")  

Remember the command to insert a new line?  That would be ```ESC shift + o```.  The rest of creating a shell script is the same as you have been doing on the command line.  The shell script will execute lines in sequential order allowing you to chain commands together.  Let's type some commands to display the last 10 lines of the hosts.deny file provided from chapter 6 and add in a message to the user.

```bash
#!/usr/bin/bash

echo "Here is the content of the ~/Documents/hosts.deny file"
echo "********************************************************"
tail ~/Linux-text-book-part-1/files/Chapter-07/logs/hosts.deny
echo "********************************************************"
```

Now we need to save the file (w) and quit out of vi (q).  You can move to __ex__ mode by hitting ```ESC :wq``` to save and quit.   Now let us run our shell script.  Type ```list-ip.sh``` on the command line.   What happens? Why?

![*Command not found*](images/Chapter-07/editors/bash/command-not-found.png "Command Not Found")

### System Path

The file is correctly named but we have a problem.  The system only knows about command binaries in certain locations.  It doesn't know about our user created binaries.  How does the operating system know where to look?  Simple, type ```echo $PATH``` and what do you see?

![*echo $PATH*](images/Chapter-07/editors/bash/system-path.png "System Path")

There is a system variable named $PATH that is constructed upon boot.  It includes the default locations that the essential command binaries, additional command binaries, and user install binaries are located.  Every time you execute a command, the system parses the command name and looks down this path to try to find the corresponding binary.  Note the absolute paths are chained together with colons ```:```. When the shell parser finds the first occurrence--it passes that location and executes that matching binary name.  In our case the shell script ```list-ip.sh``` is located in ```~/Documents``` which is not in the system path listed in the image above.  So how can we reference it?   Remember the single-dot operator ```./```--that tells the operating system to look here--overriding the system path.  Try and type ```./list-ip.sh``` what happens now?

![*Permission Denied!*](images/Chapter-07/editors/bash/permission-denied.png "Permission Denied")

### Changing Permissions for Execution

The error message tells us that we have ```permission denied```.  Remember back to chapter 6 when we dealt with file permissions? In order for a shell script to be executable we need to give it __execute__ permission.

![*ls -l list-ip.sh*](images/Chapter-07/editors/bash/permissions.png "Permissions")

We can change permissions by using the ```chmod``` command.  What is the current numeric value of the permissions for the file list-ip.sh?  What would we need to change it to at a minimum?  If you said 764 you would be correct?  Why?  The minimum we need to do is add the __execute__ permission to the owner's permission section.
We would go from this ```rw-rw-r--``` to this ```rwxrw-r--```.  We could do that by typing ```chmod 765 list-ip.sh```.  There is an easier way with group and letter shortcuts.

: Permission Shortcuts

    Group             Description
-------------   -----------------------------------
     u               Owner of the file
     g               Group owner of the file
     o               Other (everyone else)
-------------   -----------------------------------

: Permission can be added or negated and combined

    Group                                            Description
------------------------------   -----------------------------------------------------------
```chmod u+x list-ip.sh```              Gives the owner of the file execute permission
```chmod g+x list-ip.sh```            Gives the group owner of the file execute permission
```chmod o+x list-ip.sh```         Gives the other group execute permission (everyone else)
```chmod u-x list-ip.sh```           Removes file execute permission from the group owner
```chmod o-wx list-ip.sh```          Removes write and file execute permission from other
```chmod ug+rwx list-ip.sh```         Owner and group are give rwx permissions together.
------------------------------   -----------------------------------------------------------

You will notice that in the terminal (where supported) files marked executable will turn green.  If you use the ```ls -lF``` flag you will also see that executable files will be marked with an asterisk.  Now you can finally execute your command ```./list-ip.sh``` and see the last ten lines of output from the shell script.

![*Execute Permission enabled - turns green*](images/Chapter-07/editors/bash/execute.png "Execute Permission")

## Understanding .bashrc

When your system first boots up how does it know how to define system environment variables and system PATH variables?  Part of the boot process is to read and source the ```/etc/profile``` file.  This is a system wide profile that all users accounts will inherit from this file.

Next there is local login profile.  This is additional customization added to your account after you log in via username/password.  There could be any number of files from this list in this order: ```~/.bash_profile``` or ```~/.bash_login``` or ```~/.profile``` (.profile is a hold over from the Korn shell--since bash is ksh compatible).

Then once logged in upon launching a terminal there is another set of profiles to be processed. The ```~/.bashrc``` file is processed.   The template for this file is generally located in ```/etc/bashrc``` if you want to customize or replace a ```~/.bashrc``` file.  There is one final file that you can use to modify a system environment upon logout and that is the ```~/.bash_logout``` file.

If you wanted to make modification to your $PATH variable to include a directory for the newly made shell scripts you can modify the PATH directly on the command line.  The problem is that this variable will only stay in memory for the duration of the terminal window--as soon as it is closed the PATH reverts to the one it started with.  In order to permanently modify the PATH we need to modify it in one of the profile files.  The best place to make user specific changes is in the ```~/.bashrc``` file.  

![*.bashrc*](images/Chapter-07/editors/bash/bashrc.png ".bashrc")

As you can see the file is very sparse.  There is a specific header allowing you to add user logic at the end of the file.   Let's try to add to the PATH.  When updating a shell variable we need to use the ```export``` command so that the system is aware of the new variable value.  Think of it as a *refresh* command.  In addition to the PATH variable to see all the system variables your distro sets, type ```printenv``` from the shell.

> __Example Usage:__ Type ```mkdir ~/Documents/scripts```. Now copy your ```list-ip.sh``` to this directory. Let's add this directory to our PATH in our ```~/.bashrc``` file. Finally before we edit let's print out the content of the PATH system variable so we can see our changes later.  How would you do that? Let us open our ```~/.bashrc``` file in vi.  Now move the cursor position to the bottom of the file.  Type ```ESC shift+o``` to insert a new line.   Now type ```ESC i``` to change to INSERT mode.   Type the line ```PATH=$PATH:~/Documents/scripts``` followed by a new line (vim cheats and will accept the ENTER key in addition to ```ESC SHIFT+o```) and then let's export the new variable content by typing ```export PATH```  now exit __vi__.

> __Example Usage:__ To make the changes we just made register we can do two things.  We can reboot the system so all the profiles are re-read but that is a little drastic.   A shortcut to re-read and process just the ```~/.bashrc``` file is to add a single-dot separated by a space.  Type ```. ~/.bashrc``` and then display the content of the PATH variable: ```echo $PATH``` and you should see your new addition appended to the PATH variable permanently.

## Chapter Conclusions and Review

In this chapter we learned about the vi editor and mastered its three modes.  We learned how to created a shell script with vi and how to navigate and change modes within vi.  We learned about the basic set of vi commands needed to be productive.  We learned how to create a shell script, the shortcuts to change the file permissions, and how user profiles are loaded and used.

### Review Questions

1. What are the two main representatives of stream editors?
   a. gedit and kate
   b. Nano and Joe
   c. vi and Nano
   d. vi and Emacs

2. Which family of editors came first?
   a. Screen editors
   b. Butterfly editors
   c. GUI editors
   d. Stream Editors

3. What type of editor is GNU Nano?
   a. stream
   b. text
   c. A small one
   d. file

4. Who created the vi editor?
   a. Richard Stallman, 1984
   b. Brian Fox, 1989
   c. Bill Joy, 1979
   d. Bill Joy, 1983
   e. Brian Fox, 1979

5. Which of the following sequences of the history of vi is correct?
   a. Emacs -> ed -> ex -> vi
   b. ed -> em -> ex -> vi -> vim
   c. em -> ex -> vi
   d. em -> ed -> vi -> vim

6. What are the three modes in vi?  

7. What is the key you use in vi to transition between COMMAND MODE and INSERT mode?

8. What command sequence (key) in vi will add text to the right of the current cursor position?  (just the letter)

9. What command sequence (key) in vi will move you to the beginning of the next word? (just the letter)

10. What command sequence in vi will delete a single line based on the current cursor position? (just the letters)

11. What command sequence in vi will delete 10 lines from the current cursor position? (just the numbers and letters)

12. Which command in ex mode (vi) will save the current file you are working on and exit the vi editor? (include the ":")

13. In the log file u\_ex150911.log what would be the ex command to search forward for occurrences of YandexBot? (include the forward slash)

14. Assuming your pwd is Linux-text-book-part-I and you have loaded Chapter-02 > chapter-02.md into vi, what would be the ex mode command to replace all occurrences of linux with Linux?

15. Assuming your pwd is Linux-text-book-part-I and you have loaded Chapter-02 > chapter-02.md into vi, what would be the ex mode command to replace all occurrences of Linux with GNU/Linux? (remember to escape the /)

16. Assuming the your pwd is Linux-text-book-part-I and you have loaded Chapter-02 > chapter-02.md into vi, what would be the ex mode command to remove all occurrences of the word Windows?

17. Assuming a file name topsecret.sh has a permission of 644 - what is the shortcut to give just the owner of the file additional permissions to execute the script?

18. Assuming a file named moretopsecret.sh has a permission of 757 - what is the shortcut to remove all permissions from the the **other** group?

19. What is the correct command sequence to save or write out a file in GNU Nano?
   a. ^6
   b. ^X
   c. ^O
   d. :wq

20. What is the command to display the contents of the PATH system variable on the command line?
    a. echo PATH
    b. echo $PATH
    c. echo path
    d. $PATH

### Podcast Questions

View or listen to this Podcast: [https://twit.tv/shows/floss-weekly/episodes/594?autostart=false](https://twit.tv/shows/floss-weekly/episodes/594?autostart=false "PIP Podcast page")

PIP AND THE PYTHON PACKAGE INDEX

* ~7:40 What is pip and how is it related to the Python language?
* ~9:31 What are some of the utilities that a developer who is making a Python package are going to use?
* ~11:15 If someone is getting started with Python what is the right way to install packages?
* ~12:39 Is the number of different package managers and methods of installing Python packages a benefit to the new user?
* ~13:43 Does (did) Pradyun agree with the first speaker?
* ~14:34 Where does the overview of packaging for Python live (URL)?
* ~17:52 Where has Pradyun and pip secured opensource funding from?
* ~19:00 What did Sumana do with the funding received from various companies and organizations?
* ~30:30 How does Sumana explain "DevOps"?
* ~31:37 What happens when you release software more often?
* ~36:10 What is the Hollywood portrayal of those who develop and make software and how is it different from reality of developing software?
* ~39:10 What can a short term Project Maintainer do for a software project?
* ~43:20 How many maintainers did the pip project have in 2017? How many do they have now in 2020? Are they paid developers?
* ~44:44 If pip is one of the most important tool chains in the industry -- who funds its development/maintenance?
* ~46:40 What is the programing language that Python is written in?
* ~53:10 What is the tip for what not to do with pip on Linux?
* ~55:15 What is the website (URL) to keep up with Python language announcements?

Book mentioned in the Podcast:

* [The Bug by Ellen Ullman](https://www.amazon.com/dp/B00AZ181TQ/ref=dp-kindle-redirect?_encoding=UTF8&btkr=1 "The Bug by Ellen Ullman book purchase website")

### Lab Chapter 7

Objectives

The objective of this lab is to master vi commands and shell scripts

Outcomes

At the end you will have mastered the basics of vi and now be proficient in the tools of Linux shell scripting

#### Prerequisites

* You will need an additional virtual machine with Ubuntu Server 20.04 installed for this entire lab
* You will need to make sure the `vim` program is installed
* You will need to make sure the the `nano` program is installed
* You will need to clone the Textbook source code to the Ubuntu Server virtual machine in the home directory
* You will need to install the program ```vimtutor```
  * On Ubuntu by typing ```sudo apt-get install vim vim-runtime vim-gtk```
  * On Fedora by typing ```sudo dnf install vim vim-enhanced```
  * There is a good text explanation of each of the vim tutor exercises: [https://www.systutorials.com/vim-tutorial-beginners-vimtutor/](https://www.systutorials.com/vim-tutorial-beginners-vimtutor/ "vimtutor exercises")

1) Using Ubuntu Server, type the command ```vimtutor``` from the terminal. __Warning:__ ```vimtutor``` requires you to read the instructions carefully!
    i) This is a 6 part tutorial.  You need to follow all the steps of the 6 part tutorial making your changes directly in the file.
    i) __Be careful__ to save the file to an external location – otherwise IT WILL BE OVERWRITTEN each time you launch the vimtutor command. You can do this by typing ```:w  ~/Documents/vimtutor.txt``` - this way you can edit the file on your local system instead of launching the vimtutor application again.  Note you need to use ```vim``` for this assignment.
    i) Take a screenshot as you complete each sub-section (i.e. 2.1 2.2 3.1 4.1 )
1) From the textbook source code folder: ```files/Chapter-07/lab```, copy the file ```install-software.sh```to your home directory
    i) Using vim/ex commands, find all occurrences of ```python``` and replace them with ```python3```
    i) Save file and quit the vim editor
    i)  To test your work, give the shell script execute permission and execute it by using `sudo ./install-software.sh`
1) From the textbook source code folder: ```files/Chapter-07/lab```, copy the file ```install-software.sh```to your home directory
    i) Using vim/ex commands, find all occurrences of ```python``` and replace them with ```python3```
    i) Save file and quit the vim editor
    i) To test your work, give the shell script execute permission and execute it by using `sudo ./install-software.sh`
1) In your home directory, using vim, create a shell script named ```first-shell.sh``` in your home directory that contains the following:
    i) Add the proper *shebang* on the first line.
    i) Add two lines of space
    i) Store the output of the command ```date``` into the shell variable named **DT**
    i) Add the command that will print out the text: "#############"
    i) Add the command that will print out the text: "Shell successfully execute at: $DT"
    i) Add the command that will print out the text: "#############"
    i) Save the file and quit the vim editor
    i) Execute the command to give first-shell.sh execute permission
    i) Take a screenshot of the output executing first-shell.sh
    i) Take a screenshot of the command used to print the content of the file: first-shell.sh
1) From the textbook source code folder: ```files/Chapter-07/lab```, copy the file ```install-software.sh``` to your home directory
    i) Open the file install-software.sh in GNU Nano
    i) On line 5 replace my name with yours - take a before and after screenshot
    i) Save the file and quit Nano, then execute the command to show only the **first** 10 lines of the file install-software.sh
1) Using wither vim or Nano:
    i) Create a shell script named **install-textbook-dependencies.sh** in your home directory.
    i) Add the proper *shebang* on the first line, then two lines of space
    i) Type the lines at the end of this assignment into your shell script
    i) Save the file and exit from your editor
    i) Give the script execute permission and execute it
    i) To test the results, `cd` into the Linux-Text-Book-Part-1 directory (clone it if you have not) and execute the the script: `./build-linux-and-macos.sh`  (the script already have execute permission)
    i) To test if the textbook built correctly - cd into the directory: **output/pdf**.  Issue the `ls` command and you will see two PDF files.

```bash

# These lines retrieve the pandoc .deb file which is the executable that turns markdown
# into a PDF and ePub
wget https://github.com/jgm/pandoc/releases/download/2.15/pandoc-2.15-1-amd64.deb
sudo dpkg -i pandoc-2.15-1-amd64.deb

sudo apt-get install -y texlive texlive-latex-recommended texlive-latex-extra  
texlive-fonts-recommended texlive-fonts-extra texlive-xetex texlive-font-utils  
librsvg2-bin texlive-science-doc texlive-science

wget http://packages.sil.org/sil.gpg
sudo apt-key add sil.gpg
sudo apt-add-repository -y "deb http://packages.sil.org/ubuntu/ $(lsb_release -sc) main"
sudo apt-get update
sudo apt-get -y install fonts-sil-charis

sudo apt-get -y install fonts-inconsolata
sudo fc-cache -fv

```

__Deliverable:__  

Submit your GitHub URL for your repo to Blackboard.

#### Footnotes

[^77]: <a href="https://commons.wikimedia.org/wiki/File:Bill_joy.jpg#/media/File:Bill_joy.jpg">Bill joy</a> by Original uploader was <a title="en:User:SqueakBox" class="extiw" href="//en.wikipedia.org/wiki/User:SqueakBox">SqueakBox</a> at <a class="external text" href="http://en.wikipedia.org">en.wikipedia</a> - Transferred from <a class="external text" href="http://en.wikipedia.org">en.wikipedia</a>; Transfer was stated to be made by <a title="User:Jalo" href="//commons.wikimedia.org/wiki/User:Jalo">User:Jalo</a>.. Licensed under <a title="Creative Commons Attribution 2.0" href="http://creativecommons.org/licenses/by/2.0">CC BY 2.0</a> via <a href="https://commons.wikimedia.org/wiki/">Commons</a>.

[^78]: [http://www.gnu.org/software/emacs/](http://www.gnu.org/software/emacs/ "GNU Emacs")

[^79]: <a href="https://commons.wikimedia.org/wiki/File:Emacs_Dired_buffers.png#/media/File:Emacs_Dired_buffers.png">Emacs Dired buffers</a> by Emacs development team - <a title="User:Ferk (page does not exist)" class="new" href="//commons.wikimedia.org/w/index.php?title=User:Ferk&amp;action=edit&amp;redlink=1">Ferk</a> (user who took this screenshot). Licensed under <a title="Creative Commons Attribution-Share Alike 3.0" href="http://creativecommons.org/licenses/by-sa/3.0/">CC BY-SA 3.0</a> via <a href="https://commons.wikimedia.org/wiki/">Commons</a>.

[^80]:[https://en.wikipedia.org/wiki/GNU_Emacs](https://en.wikipedia.org/wiki/GNU_Emacs "GNU Emacs"

[^81]: "<a href="https://commons.wikimedia.org/wiki/File:KB_Terminal_ADM3A.svg#/media/File:KB_Terminal_ADM3A.svg">KB Terminal ADM3A</a>" by No machine-readable author provided. <a title="User:StuartBrady" href="//commons.wikimedia.org/wiki/User:StuartBrady">StuartBrady</a> assumed (based on copyright claims). - No machine-readable source provided. Own work assumed (based on copyright claims).. Licensed under <a title="Creative Commons Attribution-Share Alike 3.0" href="http://creativecommons.org/licenses/by-sa/3.0/">CC BY-SA 3.0</a> via <a href="https://commons.wikimedia.org/wiki/">Commons</a>.

[^82]: [https://en.wikipedia.org/wiki/Vi#Distribution](https://en.wikipedia.org/wiki/Vi#Distribution "vi is ex")

[^83]: [https://en.m.wikipedia.org/wiki/Vi#Ports_and_clones](https://en.m.wikipedia.org/wiki/Vi#Ports_and_clones "VI")

[^84]: [https://en.wikipedia.org/wiki/Elvis_\(text_editor\)](https://en.wikipedia.org/wiki/Elvis_(text_editor) "elvis")

[^85]: [https://en.wikipedia.org/wiki/Vi#Ports_and_clones](https://en.wikipedia.org/wiki/Vi#Ports_and_clones "nvi")
