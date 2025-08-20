# Yutovo project
Yutovo is a powerful calculator with graphical representation of mathematics operations inside a text editor.

Yutovo logger is a part of the Yutovo project, which aims logging info of other parts of the project.

## Building for Ubuntu

Install requirements:

```
sudo apt update && sudo apt install -y git cmake g++ pkg-config libspdlog-dev
```
Create the following directories:

```
mkdir -p yutovo/deploy
```
Set this variable to the yutovo directory:

```
export YUTOVO_DEPLOY=~/yutovo/deploy
```

Clone the project in the yutovo dir (select another branch if you want):

```
cd yutovo
git clone -b develop https://github.com/denprog/yutovo-logger.git
```
Create the build directories and build the debug version:

```
mkdir -p build/debug
cd build/debug
cmake -DCMAKE_BUILD_TYPE=Debug ../..
make -sj && make install
```
