```shell
cd C:\
md .\src\vcpkg
cd .\src\vcpkg
git clone https://github.com/microsoft/vcpkg
.\vcpkg\bootstrap-vcpkg.bat
set VCPKG_HOME=C:\src\vcpkg\vcpkg
set path=%path%;%VCPKG_HOME%\;
vcpkg help
```
