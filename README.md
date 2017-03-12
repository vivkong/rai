# RAI Client [![Build Status](https://travis-ci.org/rai-project/rai.svg?branch=master)](https://travis-ci.org/rai-project/rai)



## Download Binaries

The code is continuously built and published. The client can be downloaded from the following URLs (depending on your OS and Architecture):

| Operating System | Architecture | Stable Version Link                                                              |
| ---------------- | ------------ | -------------------------------------------------------------------------------- |
| Linux            | i386         | [URL](https://files.rai-project.com.s3.amazonaws.com/dist/rai-linux-386/rai)     |
| Linux            | amd64        | [URL](https://files.rai-project.com.s3.amazonaws.com/dist/rai-linux-amd64/rai)   |
| Linux            | armv5        | [URL](https://files.rai-project.com.s3.amazonaws.com/dist/rai-linux-armv5/rai)   |
| Linux            | armv6        | [URL](https://files.rai-project.com.s3.amazonaws.com/dist/rai-linux-armv6/rai)   |
| Linux            | armv7        | [URL](https://files.rai-project.com.s3.amazonaws.com/dist/rai-linux-armv7/rai)   |
| Linux            | arm64        | [URL](https://files.rai-project.com.s3.amazonaws.com/dist/rai-linux-arm64/rai)   |
| OSX/Darwin       | i386         | [URL](https://files.rai-project.com.s3.amazonaws.com/dist/rai-darwin-386/rai)    |
| OSX/Darwin       | amd64        | [URL](https://files.rai-project.com.s3.amazonaws.com/dist/rai-darwin-amd64/rai)  |
| Windows          | i386         | [URL](https://files.rai-project.com.s3.amazonaws.com/dist/rai-windows-386/rai)   |
| Windows          | amd64        | [URL](https://files.rai-project.com.s3.amazonaws.com/dist/rai-windows-amd64/rai) |


## Building From Source

This is not recommended unless you are interested in developing and/or deploying `rai` on your own cluster. To build from source simple run

```bash
go get -u github.com/rai-project/rai
```

You will need an extra secret key if you build from source.

## Usage


To run the client, use

```bash
rai -p <project folder>
```

From a user's point a view when the client runs, the local directory specified by `-p` gets uploaded to the server and extracted into the `/src` directory on the server. The server then executes the build commands from the `rai_build.yml` specification within the `/build` directory. Once the commands have been run, or there is an error, a zipped version of that `/build` directory is available from the server for download.

The server limits the task time to be an hour with a maximum of 8GB of memory being used within a session. The output `/build` directory is only available to be downloaded from the server for a short amount of time. Networking is also disabled on the execution server. Contact the teaching assistants if this is an issue.

#### Other Options

```
  -c, --color         Toggle color output.
  -d, --debug         Toggle debug mode.
  -p, --path string   Path to the directory you wish to submit. Defaults to the current working directory. (default "current working directory")
  -v, --verbose       Toggle verbose mode.
```

On Windows, it might be useful to disable the colored output. You can do that by using the `-c=false` option


## Setting your Profile

Each student will be contacted by a TA and given a secret key to use this service. Do not share your key with other users. The secret key is used to authenticate you with the server.

The `RAI_SECRET_KEY`, `RAI_TEAM_NAME`, and `RAI_ACCESS_KEY` should be specified in your `~/.rai_profile` (linux/OSX) or `%HOME%/.rai_profile` (Windows -- for me this is `C:\Users\abduld\.rai.profile`) in the following way.

```yaml
profile:
  firstname: Abdul
  lastname: Dakkak
  username: abduld
  email: dakkak@illinois.edu
  access_key: XXXXXXXXXXXXXXXXXXX
  secret_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```


## Project Build Sepecification

The `rai_build.yml` must exist in your project directory. In some cases you may not be able to execute certain builtin bash commands, in this senario the current workaround is to create a bash file and insert the commands you need to run. You can then execute the bash script within `rai_build.yml`.

The `rai_build.yml` is written as a [Yaml](http://yaml.org/) ([Spec](http://www.yaml.org/spec/1.2/spec.html)) file and has the following structure.

```yaml
rai:
  version: 0.2 # this is required
  image: nvidia/cuda:8.0-devel-ubuntu16.04 # nvidia/cuda:8.0-devel-ubuntu16.04 is a docker
                         				   # image which can be viewed at https://hub.docker.com/r/nvidia/cuda/
                         				   # You can specify any image found on dockerhub
resources:
  gpus: 1 # tell the system that you're using a gpu
commands:
  build:
    - echo "Building project"
    # Use CMake to generate the build files. Remember that your directory gets uploaded to /src
    - cmake /src
    # Run the make file to compile the project.
    - make
    # here we break the long command into multiple lines. The Yaml
    # format supports this using a block-strip command. See
    # http://stackoverflow.com/a/21699210/3543720 for info
    - >-
      ./mybinary -i input1,input2 -o output
```

Syntax errors will be reported and the job will not be executed. You can check if your file is in a valid yaml format by using tools such as [Yaml Validator](http://codebeautify.org/yaml-validator).


## Profiling

Profiling can be performed using `nvprof`. Place the following build commands in your `rai-build.yml` file

```yaml
    - >-
      nvprof --cpu-profiling on --export-profile timeline.nvprof --
      ./mybinary -i input1,input2 -o output
    - >-
      nvprof --cpu-profiling on --export-profile analysis.nvprof --analysis-metrics --
      ./mybinary -i input1,input2 -o output
```

You could change the input and test datasets. This will output two files `timeline.nvprof` and `analysis.nvprof` which can be viewed using the `nvvp` tool (by performing a `file>import`). You will have to install the nvvp viewer on your machine to view these files.

_NOTE:_ `nvvp` will only show performance metrics for GPU invocations, so it may not show any analysis when you only have serial code.

## Reporting Issues

Please use the [Github issue manager] to report any issues or suggestions.

Include the outputs of

```bash
rai version
```

as well as the output of

```bash
rai buildtime
```

In your bug report. You can also invoke the `rai` command with verbose and debug outputs using

```bash
rai --verbose --debug
```


[github issue manager]: https://github.com/rai-project/rai/issues



