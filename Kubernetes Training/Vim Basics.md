## Step 1: Intro

Vim is a highly configurable text editor built to make creating and changing any kind of text very efficient. It is included as "vi" with most UNIX systems and with Apple OS X.

![](https://opensource.com/sites/default/files/uploads/1_xkcdcartoon.jpg)

As vim is present in most linux and unix systems it is very popular still some things are not that obvious if you use vim for the first time.

Vim is rock stable and is continuously being developed to become even better. Among its features are:

- persistent, multi-level undo tree
- extensive plugin system
- support for hundreds of programming languages and file formats
- powerful search and replace
- integrates with many tools

This tutorial should help you doing basic things in vim such as:

- creating new files
- copy - paste
- search pattern
- search and replace

## Step 2: Creating new files

Creating new files is not that hard. The only thing that you have to type is:

vim first-file

_Click to run on **node1**_

Now vim is started and waits for us to tell it what we want to do. By default vim just views the file which in our case is empty. If we want to add content to the file, we first need to tell vim, that we want to edit our file.

Usually this is being done by just typing `i`.

Once this is done, you can see on the left bottom that we are in insert mode:

-- INSERT -- 

Now that we are in insert mode, we can add some content to our file.

Try to copy and paste the following content into your file:

apple
banana
orange
yellow
green
blue
red

After inserting the text we need to tell vim, that we are finished editing the file. This is done by using `ESC`.

Nothing that we have done so far acutally created or changed the file. The current state of the file just exists in our editor until we save it.

Vim has various commands that we can use to interact with the editor. They mostly start with typing `:`.

In case we want now want to save the file, we just type:

:w

_Click to run on **node1**_

`:w` is short for write. You could have also used the command `:write` with the same result.

Vim will then tell you what it did:

first-file" [New] 14L, 84C written 

We still have our editor open because we did not tell it that we want to close the edit.

If we want to close the editor, we can do this using the quit command like this:

:quit

_Click to run on **node1**_

This again also could be shorter by using just `:q`.

If you know that you want to write the contents to a file and also would like to close the editor after writing the file to disk, you can combine both commands by just typing `:wq`

## Step 3: copy - paste - delete

In this step we will use the copy paste and delete capabilities of vim to manipulate our file.

Lets use the file that we created in the last step:

vim first-file

_Click to run on **node1**_

Vim has many different commands mapped to your keyboard:

![](https://linuxhandbook.com/content/images/2021/03/vim-graphical-cheat-sheet.png)

Lets try to copy and paste all the content.

To do so press `gg` to get to the top of the file. Then press `y` followed by an uppercase `G`.

If you have a look at the cheat sheet you will se `y` stands for yank and the G stands for "go to end of file".

Now lets paste the copied content at the end of the file by typing `G` followed by `p`.

Now lets save the file:

:w

_Executed on **node1**_

If you want to delete a single line you can do so by just typing `dd`. If you would like to delete more lines you can do `d5d` to delete the next 5 lines.

In the end lets now close the file without saving the changes like this:

:q!

_Executed on **node1**_