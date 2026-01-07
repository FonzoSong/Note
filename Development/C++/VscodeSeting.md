### 只推荐在学习中使用vscode

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "C/C++: Compile all files to object files",
      "type": "shell",
      "command": "g++",
      "args": [
        "-std=c++17",
        "-Wall",
        "-c",
        "${workspaceFolder}/src/*.cpp",
        "-o", "build/"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": ["$gcc"],
      "detail": "Compile all .cpp files into object files (.o"
    },
    {
      "label": "C/C++: Link object files into executable",
      "type": "shell",
      "command": "g++",
      "args": [
        "build/*.o",
        "-o",
        "${workspaceFolder}/bin/${workspaceFolderBasename}"
      ],
      "group": "build",
      "problemMatcher": ["$gcc"],
      "detail": "Link all object files into final executable"
    }
  ]
}
```

