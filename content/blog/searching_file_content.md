+++
title = "Searching file contents for pattern"
date = 2022-04-21
description = "Creating a Python script to search a directory trees files for content"
in_search_index = true
[taxonomies]
tags = ["python"]
+++
## Intro
There have been situations where I want to find a file containing a specific fraze or a setting. Yet looking through every file can take a lot of time. Since we are programmers, let's code a simple script to do this task.

## Reading directory tree
First thing first, let's start by walking through the directory. This is easiely accomplished by using pythons `os.walk` function. It will iterate through the directory, including the subdirectories.
```python
def walk_dir(dir, term):
    for path, dirs, files in os.walk(dir):
        for file in files:
            print(path + '/' + file)

if __name__ == '__main__':
    walk_dir(sys.argv[1], sys.argv[2])
```

## Readable files
Not every file can be read as a text file. So let's make sure it's readable by checking it's extention. We will store the readable extentions in a public array.
```python
searchable = [
        ".py",
        ".java",
        ".txt",
        ".md",
        ".org",
        ".xml",
        ".json",
        ".toml",
        ".yoml",
        ".sh",
]
```
## Searchig for the pattern
Every programming language has suppor for regular expressions. In python, that module is called `re`
Now that we have the paths to each file let's create a function that returns a boolean, whether or not the file is readable.
```python
def readable(file):
    for extention in searchable:
        if re.search(extention, file):
            return True;
    return False
```
With all of the preperations done, let's create the final function, with takes in a file path, and a search term, and prints out all the path to the file, and the line where the pattern was found.
```python
def read_file(file, term):
    if not readable(file):
        return
    with open(file, 'r') as f:
        for i, line in enumerate(f.readlines()):
            if re.search(term, line): print(f"Found in '{file}'\n{i}: {line}")
```
