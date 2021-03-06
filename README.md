# Time Tracker CLI

This tool allows to insert time records, into the BairesDev Time Tracker; it is build in Go (Golang), you will need the Golang compailer to build it. The latest build excutable is available on the build folder, even though I recomend to build your own version from the code.

The login process works on top of the Time Tracker web application; to use the application you will need to login with your Google account, that process is simulated using the Go [Chromedp](https://pkg.go.dev/github.com/chromedp/chromedp#section-readme) package, which required that you have ***Google Chrome installed***.

To compile the application run:
```
sh build.sh
```

It will create a binary called `tt`, use this to execute it:
```
./build/tt
```

The first time that you open the application, it will create its required folders structure and it will ask you for your some configuration information like your focal point and your project.

The tool works with a REPL (Read-Eval-Print loop), where you have to send a command to perform an action, you can see the list of available commands by using:

```
tt > help
```

To use one of the commands, just send the command name, and the tool will requested you the needed arguments.

Example:

```
tt > rec

- #: bug

- Comment: issue with accounts

**** Record started! ****
```

The tool also allows the following syntax (which we call a short version):

`{command name}; {arg 1}; {arg 2}; {arg 3}...`

Example:
```
tt > rec; bug; issue with accounts

**** Record started! ****
```

The provided arguments will be filled in the required command fields.

Most of the commands works by requesting a set of fields and providing a sigle response after all the fields are provided, we call them *action commands*. Even though, there are some special commands the doesn't follow this pattern, we call them *interactive commands* (i.e. `temp add`)

Some commands like: `rec` or `add` are using templates; a template is a way to predefine the fields for some records,you can create a new template by using `temp add`.

There are some templates already created under the `build/templates/rec` folder, but you can delete them and add your own templates. Use `temp list` to see all the created templates.

Session management
==
In order to access the Time Tracker data and to be able to insert the information when you send a `commit`, the system need to log you in to the Time Tracker, this is done emulating a web session using the Google Chrome headless feature using the Chromedp go library. The first time that you login, and every time that your ***session expires***, you will be asked for your password and the login process will be executed again.
```
************************************************
** Welcome to BairedDev Time Tracker CLI tool **
************************************************

- Google password:

**** Performing Google login, it will take a while please wait... ****

**** Login successfull!! ****
```

This is the less smooth part of the application, since emulating the login in that way is slow; even though as soon as you have a valid session you can continue working without having to login again, even you can close the application during some time, and open it again without having to login again.

Workflow
==

This is the workflow which was used to design the tool:

1. Use `rec` to start a time recorder. It will request a template (you must have some created already); and, if it is required, a description; tt will record this information along with the initial time.
2. Use `view` to see your current working task.
3. Use `end` to complete your task, it will use the current time and the recored start time to calculate the worked time.
4. Repeate the process for all you day tasks.
5. Use `list` to see all your recorded time.
6. Use `commit` to send your records to the BairesDev Time Tracker.
7. Use `poure` to add the time on your pool to the current date. Se below for more details.

Edge cases
==

1. If you missed to record some time, you can use `add` to manually add it, it will request a template, an if it is required, a description and a task duration (in hours).
2. If you added a recored by mistake, you can use `delete` to remove it.
3. If you missed the start time of a task that you are currently working, use `rec at` to start it at a defined hour.
4. If you missed the end time of a task that you are currently working, use `end at` it will calculate the time base on the end hour that you provide.
5. If started the time record with the wrong template, use `edit` to change the record information, keeping the same start time.
6. If you want to edit an exiting record, you can use `edit stored` to see the record details and change it.

The tool works on today's date by default; but you can use `change date` to change to another date. This can be useful to add missing records for a previous day, you must use `add` to add time on a diffent date.

The `change date` command expect the date in the format `yy-mm-dd`, even though some other special syntaxes are allowed:
1. `change date; now`, `change date; today` and `change date;`: changes the date back to the current day.
2. `change date; yesterday`: changes the date to the previous day date.
3. `change date; {N}`: where `{N}` is an integer number, changes the date by the specified number, for example `change date; -3`, changes the date 3 days back; and `change date; 5` change the date 5 days in the future.

Batch usage
==
You might need to add multiple records at the same time, in that case you can do the following:

1. Create a file `tasks.txt` with all your commands:
```
add; bug; first task; 0.5
add; feat; second task; 1
add; meeting; third task; 2
``` 
2. Open the application and login to the it, to create a session. If you have not done the configuration process yet, you will need to do it.
```
./build/tt
```
3. Exit the aplication:
```
tt > exit
```
4. Send the commands in your file:
```
./build/tt < tasks.txt
```

Status bar
==

When you are using the application, the prompt may change adding some status values:
```
[Worked:0.25][Commited:9.00][Pool:0.25][Tracking:0.25] tt >
```

1. ***Worked***: Is the total time that you have worked during the day and that is not commited. Once you commit this time will be added to Commited.
2. ***Commited***: Is the total time that you have worked during the day that is commited.
3. ***Tracking***: is the time worked on the current task. Once you end the task this time will be added to Worked.
4. ***Pool***: is the time that could not bee commited becuase it exceed your configured daily working time. See below for more details.

Pool
=
If you exceed your daily working time limit, when you `commit` your time, the remaining time will be save in your `pool`, you can later `poure` that time on another working day.

For now, we the tool is not handling records for extra time.

Time Recording
==
By using the command `rec` you can start a timer and record your time, you will notice the tracked time on the status bar as `[Tracking:0.50]` notice that the time is tracked in quarters of an hour, also rounds to the closer quarter of hour if it is a fraction of time. So, it will show first `0.00`, in your first 7.5 minutes of work, then `0.25` between 7.5 and 22.5 minutes; then it will show `0.50` in the next 15 minutes and so on.

Application folders structure
==

The first time you open the application, it will create it own folder structure. This are the relevant files under those folders:
1. ***.tmp***: it stores temporary files like the application state and a cache.
2. ***local***: it a folders base local db, to store the time records before commiting them, the records are store here on `.json` format.
3. ***templates***: commands like `rec` or `add` uses templates (see above); those templates are saved here on `.json` format. The template fields that starts with "x-" are information fields, which means that they don't store values required by the template.

Besides that, you will see a `config.json` file, which stores your email, project, focal point, and working time information, all this information is required when you first run the application. If this file is removed, all the information will be required again when run the application.
