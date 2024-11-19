Template project for STM32F401CCUx
## Usage
To setup:
```bash
make cmake
```
To build:
```bash
make build
```
To flash with st-tools:
```bash
make flash
```
## Project name
Name for the project is taken automagically from current dir. 
Can change the name in both `CMakeLists.txt` and `Makefile`
## CubeMX
When regenerating the project, tick `Copy only the necessary library files` in Code Generator tab.
Cube doesn't seem to rewrite Makefile, when regenerating
