# Intro to Linux/bash 

*by Maria Eleonora Rossi and Iker Irisarri*

In this tutorial, we are going to use the command line for our exercise. The command line is a text-based interface used widely in bioinformatics to interact with computers.
A bit of familiarity with the command line will help you greatly in our practical; however, here is a little refresher on the commands that are going to help you navigate folders, read files, and launch analyses.

We start the tutorial in the Terminal. Open the Terminal on your machine. You'll likely see something like this
```sh
(base) eleonora@mac ~ % 
```
The `%` is called the prompt; on other machines (e.g. Windows) can also look like this `>`.

## Navigating and working in the filesystem

The nested hierarchy of folders and files on your computer is called the filesystem. When we use the computer, we are used to clicking on the folder or the file we want to open. On the terminal, we cannot do that with a click, but we need to navigate and access the folder (we normally refer to them as directories) we want to open with specific commands. 

All directories are nested within a previous directory, and you can also have multiple directories in the same location, as you would have, for example, different folders in your Finder app. 

To know where we are in the filesystem of our computer, we can type in the directory `pwd`, short for print working directory.

```sh
(base) eleonora@mac Desktop % pwd
/Users/eleonora/Desktop
```
`/Users/eleonora` is the pathway, and `Desktop` is the directory I am in. Each directory is separated by `/` symbol.

Let's say we want to create a new directory, normally we would do right click -> new. Here we use the command `mkdir` (make directory).

```sh
(base) eleonora@mac Desktop % mkdir Linux_tutorial
```
To check if we have created the directory, we type `ls` (list), which allows us to see all the files and directories present in the directory we're in.

```sh
(base) eleonora@mac Desktop % ls
Linux_tutorial
```
Now we want to go into the directory we just created. We can use `cd` (change directory) and the name of the directory

```sh
(base) eleonora@mac Desktop % cd Linux_tutorial 
(base) eleonora@mac Linux_tutorial % 

```
Type again `pwd` and check the pathway you're in. Is it similar to the previous pwd?

If we want to exit the directory we are in, we need to 'move back' in the folder. The wildcard to do is `..`, the full command is `cd ..`. This command always moves you one directory back. If you want to move 2 directories back, you just need to type `cd ../../`. Remember to delimit each directory with the `/`. 

Type `cd ..`
What is the path now?

Let's create another directory in `Linux_tutorial` and call it `data`. What would you type?
Enter the directory and check the absolute path.

Ok now we have the basics to navigate the filesystem. Let's see how to work with files.

## Working with files

Most times, we need to visualise the files. To do so, there are different commands we can use. `less` is very useful; it loads the file you want to visualise, and you can "scroll" the file by hitting the space bar or actually scrolling with the mouse. The advantage of `less` is that it loads in the memory temporarily only the section of the file you are looking at, meaning that it's handy to look at very large files. 

In the phylogenomics tutorial, at one point, you will have to visualise the `chicken.faa` file. Navigate to the directory and use `less` to view it.

Example
```
less chicken.faa
```
What is in the file? Try to scroll the file. To exit just press `q`.

Two other useful commands are `head` and `tail` 
As the names might suggest, `head` shows you the first part of the file. With the flag `-n`, you can specify how many lines you want to see of the file. 
E.g. if you type `head -n 30 chicken.faa` you will visualise the first 30 lines of that file.

A similar concept can be applied to `tail`, where it allows you to check the last part of the file. 
What would you type to check the last 20 lines of the `chicken.faa` file?

## Editing files

The are several text editors you can use to edit a file or a script. One of the easiest to use in `nano`. 

```
nano <name-file>
```
Once opened, you can look at it, you can move around with the cursor, write, paste, copy, etc, etc. Once done, you can ext `nano` with `ctrl + x`. If the file has been modified, `nano` will ask if you want to save it, and you say yes by typing `y`. The program will then quit, and you will find the file in the directory. 

## Running a program

All the programs that you have installed on your machine or that you'll find in the tutorial have a manual. To access the manual, you can type the name of the program followed by the flag `-h` or `--h` or `-help`. This will show you, on the screen, all the possible flags and options you can execute the command with.


These are just some very basic commands and instructions on how to use the command line. Please come back to this tutorial if you need help while running the phylogenomics practical.

For a more in-depth read, a very good starting book for your first steps in bioinformatics is *Practical Computing for Biologists by Haddoc and Dunn*. You can find more info [here](https://practicalcomputing.org/index.html). 

Finally, here is the [cheat sheet](https://linuxstans.com/bash-cheat-sheet/) for some basic bash commands.
 
Good luck and have fun!
