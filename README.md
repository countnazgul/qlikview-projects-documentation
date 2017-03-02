# QlikView Project Structure

### Step based build?

Recently I was playing with IPython Notebook and got myself thinking if this approach can be applied to QlikView development somehow. I’ve spend some time thinking and testing and can share (and looking for feedback) the initial prototype structure and logic.


### Workflow

The idea behind the step loads is that the current step is always loading binary from the previous step. This will reduce the reloads waiting time because we are going to wait only for the binary load (fast) + the current step script and not the whole script from the beginning to the end (imagine to wait few minutes instead of 1 hour before realise that there is a missing comma at the last part of the script). Of course if change is made in the first step then all other steps have to be reloaded and this time the overall wait time will be bigger since each step has to be saved to the disk and then loaded from the next step.


### Summary
The whole approach can be separated in 3 parts:

1. Folder structure
2. Config
3. Helper
4. Build

### Solution Folder Structure

The top level folder structure is:
```
root/
├── _config/
├── bin/
├── build/
└── src/
    ├── data/ 
    ├── qvw/
    └── scripts/
```

* **_config** - contains solution configuration files, global scripts
* **bin** - small console application which will be used to maintain the solution
* **build** - final qvw files
* **src** - where all the magic is done

## "Src" Folder

Folder structure:

```
├── ...
├── src/
│   ├── data
│   │   ├── Project1/
│   │   │   ├── Project1_Step1/
│   │   │   │   ├── data1.qvd
│   │   │   │   ├── data2.qvd
│   │   │   │   └── ...
│   │   │   └── Project1_Step2/
│   │   │       └── ...
│   │   └── Project2/
│   ├── qvw
│   │   ├── Project1/
│   │   │   ├── Project1_Step1.qvw
│   │   │   ├── Project1_Step2.qvw
│   │   │   └── ...
│   │   ├── Project2/
│   │   └── ...
│   └── scripts
│        ├── Project1/
│        │   └── Project1_Step1/
│        │       │── 01Mappings.qvs
│        │       │── 02InlineLoads.qvs
│        │       └── 03Extract.qvs
│        │   └── Project1_Step2/
│        │       │── 01Transform.qvs
│        │       └── 02Store.qvs
│        ├── Project2/
│        └── ...
````

`src` folder is where everything is happening. `src` folder structure can contain multiple projects and each project can contain multiple steps. Each step is represented as a single qvw in the `qvw` project folder and each step load `binary` the qvw from the previous step. All the scripts are written in the `scripts` folder again split by project and steps and each step can have more than one script file (same as the script tabs in QV desktop)

The script files can be edited in any text editor.

##### "qvw" Folder

This folder contains all qvw files (separated by project). Only one line of script is added. This script line will `Must_include` a script from the `_config` folder which will binary load the previous step, load all `qvs` scripts for this step and load the global script file (also from `_config` folder)

## "bin" Folder

Here will be a small console app that will help with adding/removing project, adding/removing step, build project etc.

## "_config" Folder

Few files will be stored here. 

* solution config - YAML file which will hold global configuration options like
  * qv.exe path (will be used in the helper exe in the `bin` folder
  * generate `-prj` folders for each step
  * various of paths (will be used during the build process)
* global script - QV script that will automatically be loaded in every step. Can be used to load globally (for the solution) external paths, connection strings etc.

## "build" Folder

All final qvw files. The qvw files here will be produced during the build process.

## Build process

This process will combine all steps script files (for a project) into one, will copy the last step qvw (into `build` folder) and will add the full script into it. 

During the build process, using the YAML file (in `_config` folder) all local variables can be replaced to match the production folder structure. For example if we have variable `sDataFolder` which point to the project data folder in the solution during the build process this variable can be replaced with the actual path in the `SourceDocs` folder on the server. Also we can have different paths for the different environments (test and prod). This replacement process can help us to change the paths quickly if the prod folders are changed - no need to hunt where the full paths are used - just change the paths in the config file and build.
