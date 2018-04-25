# Becoming a Unix Power User

Your entire life you've been using your computer's GUI (graphical user
interface). But the GUi is a relatively modern innovation. Before PCs became
popular most computers were based on a text interface called the terminal (or
command prompt in Windows). Even today, the terminal is a very popular tool
among computer scientists and software developers since using the terminal can
greatly improve your efficiency. 

The terminal is a very complicated tool - here we'll just go over the basics
. 

The unix file system is built like a tree. Everything is contained in a "root"
directory - below that is a lot of system files and somewhere within all of
those are your user specific files. 

```
                              root (/)
                    /       |         |           \
                   usr      bin      var          ...
                  /   \
                 You  Other
                  |
             all your files
```

The biggest challenge with using a Unix terminal is figuring out how to
efficiently traverse the file system. When you open the terminal app on your 
Mac it starts out at your users "Home" directory. This will contain your
Desktop, Documents, etc folders. To change
folders type `cd <folder>`. 
For example, to change to the Desktop type:

```Bash 
~ $ cd Desktop
```

When you're in some directory, `.` refers to the current directory. `..` 
refers to the parent directory, `../..` refers to the gradparent directory. So
to go up back one directory type `cd ..`. 

You can use the `ls` command to list the files and directories in any directory.
`ls`
will list the (non-hidden) files and directories in the current directory. 
`ls -a` 
will list all the files and directories in the current directory.
`ls <folder>`
will list all files and subdirectories within the specific folder. 

Now that you have the basics of moving around the file system here are some
- `$ pwd` : Prints where you are in your file directory.
- `$ ls` : Prints the contents of whichever folder you're currenly in.
- `$ ls -a` : Prints the contents of whichever folder you're currently in
including hidden files and folders. In windows this command is \$ dir -a.
- `$ cd <location>` : Moves to the specific location. For example if you're in 
	a folder with a subfolder named "next", \$ cd next will move you into the 
	next folder. 
- `$ ~` : Referes to the home directory.
- `$ .` : Refers to the current directory.
- `$ ..` : Reffers to the parent directory. 
        \item \$ ...\\Refers to the grandparent directory.
        \item \$ /\\Refers to the root directory. 
        \item \$ rm <file>\\Removes a file or empty folder.
        \item \$ rm -rf <item>\\Recursively removes a file or non-empty folder (used to delete folders with lots of files and subfolders).
        \item \$ cp <source location/filename> <sink location/optional new filename>\\Copies a file from one location to another (if a new filename is provided the file will also be renamed).
        \item \$ mv <source location/filename> <sink location/new filename>\\Moves a file from one location to another. Can also be used to rename a file if you move a file to it's currentl location but specify a new name. 
        \item \$ vim <file>\\Opens a file in vim (a terminal based text editor). You can also use nano or emacs with a default unix installation.
        \item \$ sudo\\Invokes super user permissions (stands for super user do)
