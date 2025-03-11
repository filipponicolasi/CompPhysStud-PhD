# Setting Up and Running C Programs in AlmaLinux 9 (Docker Container)

## 1. Access the AlmaLinux 9 Container

If your container is already running, connect to it using:

```sh
docker exec -it <container_name> /bin/bash
```

Replace `<container_name>` with your actual container name.

If the container is stopped, start it first:

```sh
docker start <container_name>
docker exec -it <container_name> /bin/bash
```

## 2. Install Nano (Text Editor)

AlmaLinux 9 does not have `nano` pre-installed. Install it using:

```sh
dnf install -y nano
```

## 3. Install GCC (C Compiler)

To compile C programs, install `gcc` (GNU Compiler Collection):

```sh
dnf install -y gcc
```

Verify the installation by checking the version:

```sh
gcc --version
```

## 4. Create a Simple C Program

Use Nano to create a C source file:

```sh
nano hello.c
```

Inside the file, enter the following C code:

```c
#include <stdio.h>

int main() {
    printf("Hello, AlmaLinux!\n");
    return 0;
}
```

## 5. Save and Exit Nano

- Press **CTRL + X** to exit.
- Press **Y** to confirm saving.
- Press **Enter** to keep the file name `hello.c`.

## 6. Compile the C Program

Compile the program using `gcc`:

```sh
gcc hello.c -o hello
```

This command generates an executable file named `hello`.

## 7. Run the Compiled Program

Execute the program with:

```sh
./hello
```

If everything is set up correctly, you should see:

```
Hello, AlmaLinux!
```

## 8. (Optional) Install Development Tools

If you need additional development tools, install them using:

```sh
dnf groupinstall -y "Development Tools"
```

This installs tools like `make`, `gdb`, and more.

## 9. Exit the Container

To exit the container, simply type:

```sh
exit
```

## 🎉 Setup Complete!

Now you have a fully functional C development setup inside your AlmaLinux 9 Docker container! 🚀 Happy coding!
