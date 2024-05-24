# How to use cvs2svn/cvs2git with Docker

First of all this project notes that using a Docker container is advised, as the requirements can't easily be satisfied on many machines, let alone certain operating systems (Windows!).

Only requirement then is to have Docker or Docker Desktop installed - or a platform equally capable of building images and running containers.

## Build images

### Build a Docker image for cvs2svn usage

If you want to migrate from a very old SCM to an old SCM, well: build the Docker image `cvs2svn` with this command:
```
docker build --target=run -t cvs2svn .
```

### Build a Docker image for cvs2git usage

Makes much more sense. If you're hanging on an old CVS repository somewhere and want to arrive at the future, walk this way and build a `cvs2git` image:
```
docker build --target=cvs2git -t cvs2git .
```

## Using images

For *cvs2svn* usage is described in the [cvs2svn documentation](cvs2svn.md) but for *cvs2git* you have to [look elsewhere](https://www.mcs.anl.gov/~jacob/cvs2svn/cvs2git.html) - which is the base for [cvs2git.html](cvs2git.html).

### Using cvs2git for conversion from CVS to git

It is nicely documented at above links but also a bit confusing. In fact all you need is the *cvs2git* Docker image and then mix the cvs2svn-Docker instructions with the standalone *cvs2git* instructions.

So given the following:

1. you built the *cvs2git* Docker image 
2. you copied a complete CVS repository to a locally accessible directory (example: `/path/to/local/cvsrepo`)

you can start a conversion like this:
```
docker run -it --rm --mount type=bind,src=/path/to/local/cvsrepo,dst=/cvs,readonly --mount type=bind,src=/path/to/local/tmp,dst=/tmp cvs2git --blobfile=/tmp/cvs2git.blob --dumpfile=/tmp/cvs2git.dump --eol-from-mime-type  /cvs/my_cvs_repository
```

Of course this is an example only. See documented parameters or just start the container with parameter `--help`, it will gladly print out some help.

#### Volumes in use

The Docker container mounts two volumes:

1. `/cvs` containing the local CVS repository copy. \
Important: it must contain the `CVSROOT` directory, otherwise it won't work.
2. `/tmp` pointing to a local temp directory. \
Used for temporary files but also for the final output. In above example we get a blob and a dump file for later git import.

#### Hints

If your CVS repository is named `my_cvs_repository`. But modules of a CVS repository can also be migrated by using path `/cvs/my_cvs_repository/module`. This is all well documented in this tool's documentation.

Working on Windows you need to give the drive name first. So if your local CVS repository can be found at `C:\path_to\local\cvs_repository` use `src=/c/path_to/local/cvs_repository` for the mount.


### Importing dumps into a git repo

As documented [here](https://www.mcs.anl.gov/~jacob/cvs2svn/cvs2git.html) in usage step 5, use `git fast-import` to import the blob and dump file into a local git repository.

*IMHO creation of the local git repo should be done without the `--bare` parameter so you can handle it like any other repository and connect and push it to a remote repository.*

The fast-import step is a one-liner with Linux:
```
cat /path/to/local/tmp/git-blob.dat /path/to/local/tmp/git-dump.dat | git fast-import
```

and a two-liner on Windows:
```
git fast-import --export-marks=c:\temp\git-marks.dat < c:\temp\git-blob.dat
git fast-import --import-marks=c:\temp\git-marks.dat < c:\temp\git-dump.dat
```
