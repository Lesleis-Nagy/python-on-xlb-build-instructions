# Python on xlb, local build instructions.

A document detailing how to build a local version of python on the xlb machines.

# Setting up the basic environment
## optional
In order to set up a basic basic environment we first set up `.bashrc`, `.bash_profile` and `.profile`. We can use `.bashrc` as the 
'master' bash configuration file by first deleting `.bash_profile` and `.profile` and then doing
```console
$> cd
$> ln -s $HOME/.bashrc .bash_profile
$> ln -s $HOME/.bashrc .profile
```
## required
We will use environment modules to manage our python install (along with its prerequisites) and so we need to create a directory that
will hold our local modules
```console
$> cd
$> mkdir modules
```
and then editing `.bashrc` (or however we manage bash) to include
```
...
export MODULEPATH=$HOME/modules
...
```

# Building prerequisites
Before building prerequisistes, create a directory called `downloads`
```console
$> mkdir downloads
```
We will then download our three prerequisistes 
```console
$> cd downloads
$> wget https://gcc.gnu.org/pub/libffi/libffi-3.4.3.tar.gz
$> wget https://ftp.openssl.org/source/openssl-1.1.1t.tar.gz
$> wget https://www.python.org/ftp/python/3.10.11/Python-3.10.11.tgz
```
After this, we will build packages in a `build` directory
```console 
$> cd
$> mkdir build
$> cd build
```

## ffi
First we build ffi at version 3.4.3
```console
$> tar xvf ~/downloads/libffi-3.4.3.tar.gz
$> cd libffi-3.4.3
$> ./configure --prefix=$HOME/install/ffi/3.4.3
$> make
$> make install
```

Once the build is successful, we create a module file that will control this package
```console
$> mkdir ~/modules/ffi
$> nano ~/modules/ffi/3.4.3
```
We then paste the following in to this file. **WARNING** make sure to replace `YOUR_HOME_DIR` on the line that begins with `set server root` with the *full* path to your home directory
```tcl
#%Module -*- tcl -*-
##
## FFI module file
##
proc ModulesHelp { } {
    global ffi_version

    puts stderr "\tAdds FFI (version $ffi_version) to the environment."
}

module-whatis "FFI is the foreign function interface."

set ffi_version     3.4.3
set server_root     YOUR_HOME_DIR/install
set ffi_install_dir $server_root/ffi/$ffi_version

prepend-path LIBRARY_PATH $ffi_install_dir/lib64
prepend-path LD_LIBRARY_PATH $ffi_install_dir/lib64

prepend-path PKG_CONFIG_PATH $ffi_install_dir/lib/pkgconfig

prepend-path --delim " " LDFLAGS "-L$ffi_install_dir/lib64 -lffi"

prepend-path --delim " " CFLAGS "-I$ffi_install_dir/include"

prepend-path --delim " " CPPFLAGS "-I$ffi_install_dir/include"
```

Once ffi is installed, and the module file has been saved you can check the module with

```console
$> module show ffi
```

it should give something like

```
YOUR_HOME_DIR/modules/ffi/3.4.3:

module-whatis   {FFI is the foreign function interface.}
prepend-path    LIBRARY_PATH YOUR_HOME_DIR/install/ffi/3.4.3/lib64
prepend-path    LD_LIBRARY_PATH YOUR_HOME_DIR/install/ffi/3.4.3/lib64
prepend-path    PKG_CONFIG_PATH YOUR_HOME_DIR/install/ffi/3.4.3/lib/pkgconfig
prepend-path    --delim { } LDFLAGS {-LYOUR_HOME_DIR/install/ffi/3.4.3/lib64 -lffi}
prepend-path    --delim { } CFLAGS -IYOUR_HOME_DIR/install/ffi/3.4.3/include
prepend-path    --delim { } CPPFLAGS -IYOUR_HOME_DIR/install/ffi/3.4.3/include
```

You can then check that those paths exist with `ls`.

## OpenSSL

Enable the ffi module
```console
$> module load ffi/3.4.3
```
and build open ssl
```console
$> cd ~/build
$> tar xvf ~/downloads/openssl-1.1.1t.tar.gz
$> cd openssl-1.1.1t
$> ./config --prefix=/home/lesnagy/install/openssl/1.1.1
$> make
$> make install
```

Once the build is successfule we create the module file that will control this package
```console
$> mkdir ~/modules/openssl
$> nano ~/modules/openssl/1.1.1
```
We then paste the following in to this file. **WARNING** make sure to replace `YOUR_HOME_DIR` on the line that begins with `set server root` with the *full* path to your home directory
```tcl
#%Module -*- tcl -*-
##
## OpenSSL module file
##
proc ModulesHelp { } {
    global openssl_version

    puts stderr "\tAdds OpenSSL (version $openssl_version) to the environment."
}

module-whatis "OpenSSL is the secure sockets layer."

set openssl_version     1.1.1
set server_root     YOUR_HOME_DIR/install
set openssl_install_dir $server_root/openssl/$openssl_version

prepend-path LIBRARY_PATH $openssl_install_dir/lib
prepend-path LD_LIBRARY_PATH $openssl_install_dir/lib

prepend-path --delim " " LDFLAGS "-L$openssl_install_dir/lib -lssl"

prepend-path --delim " " CFLAGS "-I$openssl_install_dir/include"

prepend-path --delim " " CPPFLAGS "-I$openssl_install_dir/include"
```

Again once openssl has been installed and the module file created, we can check everything like we did with ffi.

## Python

In order to build python **make sure that ffi and openssl modules are enabled**.
```console
$> module load ffi/3.4.3 openssl/1.1.1
```

Then we can build python

```console
$> cd ~/build
$> tar xvf ~/downloads/Python-3.10.11.tgz
$> cd Python-3.10.11
$> ./configure --prefix=/home/lesnagy/install/python/3.10.11
$> make
$> make install
```

Next we create set up the modules file

```console
$> mkdir ~/modules/python
$> nano ~/modules/python/3.10.11
```

and paste the following. **WARNING** make sure to replace `YOUR_HOME_DIR` on the line that begins with `set server root` with the *full* path to your home directory

```tcl
#%Module -*- tcl -*-
##
## Python module file
##

prereq ffi/3.4.3
prereq openssl/1.1.1

proc ModulesHelp { } {

    global python_version

    puts stderr "\tAdds Python (version $python_version) to the environment."

}

module-whatis "Python is an interpreted language."

set python_version     3.10.11
set server_root     /home/lesnagy/install
set python_install_dir $server_root/python/$python_version

prepend-path PATH $python_install_dir/bin
```

And finally we can check the install using `module show python/3.10.11` as we did with ffi and openssl.

# Using the new python

In order to use the new python we should load the modules. **THIS MUST BE DONE EVERY TIME YOU LOG IN**.
```console
$> module load ffi/3.4.3 openssl/1.1.1 python/3.10.11
```

The `which` command should tell you which python you're running
```console
$> which python
```

Alternatively you can just run `python` and look at the interpreter version.

It is best to create virtual environments

```console
$> cd
$> mkdir venvs
$> python -m venv ~/venvs/my_virtual_env
```

You can then activate it

```console
$> source ~/venvs/my_virtual_env/bin/activate
```

and install whatever you need with `pip`.
