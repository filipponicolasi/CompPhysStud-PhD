# Installing and Running Python in AlmaLinux 9 (Docker Container)

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

## 2. Update Packages

Before installing Python, update the system:

```sh
dnf update -y
```

## 3. Install Python 3

AlmaLinux 9 uses **Python 3** by default. Install it with:

```sh
dnf install -y python3
```

Verify the installation by checking the version:

```sh
python3 --version
```

## 4. (Optional) Install pip

`pip` is the Python package manager. Install it with:

```sh
dnf install -y python3-pip
```

Verify the installation of `pip`:

```sh
pip3 --version
```

## 5. Test Python Installation

Create a simple Python script:

```sh
nano hello.py
```

Inside the file, write the following Python code:

```python
print("Hello, Python on AlmaLinux!")
```

Save and exit Nano (**CTRL + X**, then **Y**, then **Enter**).

Run the script:

```sh
python3 hello.py
```

You should see:

```
Hello, Python on AlmaLinux!
```

## 6. Exit the Container

If you have finished, exit the container:

```sh
exit
```

## 🎉 Setup Complete!

Python is now installed and fully functional in your AlmaLinux 9 Docker container! 🚀 Happy coding!
