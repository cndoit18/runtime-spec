# Configuration file

The container’s top-level directory MUST contain a configuration file called `config.json`.
For now the schema is defined in [spec.go](https://github.com/opencontainers/runc/blob/master/spec.go) and [spec_linux.go](https://github.com/opencontainers/runc/blob/master/spec_linux.go), this will be moved to a JSON schema overtime.

The configuration file contains metadata necessary to implement standard operations against the container.
This includes the process to run, environment variables to inject, sandboxing features to use, etc.

Below is a detailed description of each field defined in the configuration format.

## Manifest version

* **version** (string, required) specifies the version of the OCF specification which the container bundle complies with. If the container is compliant with multiple versions, it SHOULD advertise the most recent known version to be supported.

*Example*

```json
    "version": "1"
```

## Root Configuration

* **path** (string, required) Each container has exactly one *root filesystem*. The path string element specifies the path to the root file system for the container, relative to the path where the manifest is. A directory MUST exist at the relative path declared by the field.
* **readonly** (bool, optional) If true then the root file system MUST be read-only inside the container. Defaults to false.

*Example*

```json
"root": {
    "path": "rootfs",
    "readonly": true
}
```

## Mount Configuration

Additional file systems can be declared as "mounts", declared by the array element mounts. The parameters are similar to the ones in Linux mount system call. [http://linux.die.net/man/2/mount](http://linux.die.net/man/2/mount)

* **type** (string, required) Linux, *filesystemtype* argument supported by the kernel are listed in */proc/filesystems* (e.g., "minix", "ext2", "ext3", "jfs", "xfs", "reiserfs", "msdos", "proc", "nfs", "iso9660"). Windows: ntfs
* **source** (string, required) a device name, but can also be a directory name or a dummy. Windows, the volume name that is the target of the mount point. \\?\Volume\{GUID}\ (on Windows source is called target)
* **destination** (string, required) where the file system is mounted relative to the container rootfs.
* **options** (string, optional) in the fstab format [https://wiki.archlinux.org/index.php/Fstab](https://wiki.archlinux.org/index.php/Fstab).

*Example (Linux)*

```json
"mounts": [
    {
        "type": "proc",
        "source": "proc",
        "destination": "/proc",
        "options": ""
    },
    {
        "type": "tmpfs",
        "source": "tmpfs",
        "destination": "/dev",
        "options": "nosuid,strictatime,mode=755,size=65536k"
    },
    {
        "type": "devpts",
        "source": "devpts",
        "destination": "/dev/pts",
        "options": "nosuid,noexec,newinstance,ptmxmode=0666,mode=0620,gid=5"
    },
    {
        "type": "tmpfs",
        "source": "shm",
        "destination": "/dev/shm",
        "options": "nosuid,noexec,nodev,mode=1777,size=65536k"
    },
]
```

*Example (Windows)*

```json
"mounts": [
    {
        "type": "ntfs",
        "source": "\\?\Volume\{2eca078d-5cbc-43d3-aff8-7e8511f60d0e}\",
        "destination": "C:\Users\crosbymichael\My Fancy Mount Point\",
        "options": ""
    }
]
```

See links for details about [mountvol](http://ss64.com/nt/mountvol.html) and [SetVolumeMountPoint](https://msdn.microsoft.com/en-us/library/windows/desktop/aa365561(v=vs.85).aspx) in Windows.

## Processes configuration

* **terminal** (bool, optional) specifies whether you want a terminal attached to that process. Defaults to false.
* **user** (string, required) indicates the user which the process will run as.
* **cwd** (string, optional) is the working directory that will be set for the executable.
* **env** (array of strings, optional) contains a list of variables that will be set in the processes environment prior to execution. Elements in the array are specified as Strings in the form "KEY=value". The left hand side must consist solely of letters, digits, and underscores '_' as outlined in [IEEE Std 1003.1-2001](http://pubs.opengroup.org/onlinepubs/009695399/basedefs/xbd_chap08.html).
* **args** (string, required) executable to launch and any flags as an array. The executable is the first element and must be available at the given path inside of the rootfs. If the executable path is not an absoulte path then the search $PATH is interpreted to find the executable.

*Example*

```json
"process": {
    "terminal": true,
    "user": "daemon",
    "env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
        "TERM=xterm"
    ],
    "cwd": "",
    "args": [
        "sh"
    ]
}
```


## Hostname

* **hostname** (string, optional) as it is accessible to processes running inside.

*Example*

```json
"hostname": "mrsdalloway"
```

## Machine-specific configuration

* **os** (string, required) specifies the operating system family this image must run on. Values for arch must be in the list specified by the Go Language document for [$GOOS](https://golang.org/doc/install/source#environment).
* **arch** (string, required) specifies the instruction set for which the binaries in the image have been compiled. Values for arch must be in the list specified by the Go Language document for [$GOARCH](https://golang.org/doc/install/source#environment).

```json
"platform": {
    "os": "linux",
    "arch": "amd64"
}
```

Interpretation of the platform section of the JSON file is used to find which platform specific section may be availble in the document. For example if `os` is set to `linux` then the `linux` JSON object SHOULD be found in the `config.json`.

