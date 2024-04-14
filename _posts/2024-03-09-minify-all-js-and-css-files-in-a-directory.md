---
title: "Minify all JS and CSS files in a given directory, using Node.js"
date: 2024-03-09 00:01:00 +0700
categories: [Programming, Javascript]
tags: [Programming, Javascript, Scripts]
pin: false
---

Introduction
------------

In the fast-paced world of web development, optimizing performance is crucial. One effective way to enhance website load times and decrease bandwidth usage is through the minification of JavaScript and CSS files. Minification is the process of removing unnecessary characters from source code (like whitespace, comments, etc.) without changing its functionality. This results in a significantly reduced file size, which can speed up the loading process of web pages.

To assist developers in this optimization task, we introduce a comprehensive script developed using Node.js. This script automates the minification of JavaScript and CSS files across a specified directory. By leveraging powerful libraries such as uglify-js for JavaScript and clean-css for CSS, it efficiently scans, minifies, and overwrites the files in place, making the process straightforward and error-free.

This document will guide you through the details of the script's operations, including its structure, usage instructions, and error handling measures, ensuring you can deploy it effectively within your projects to achieve improved performance.

Overview
--------

This document explains the functionality of a JavaScript script designed for file system operations, specifically targeting the minification of JavaScript (JS) and Cascading Style Sheets (CSS) files in a directory. The script utilizes Node.js modules such as `fs` for file system operations, `path` for file path operations, `uglify-js` for compressing JavaScript files, and `clean-css` for compressing CSS files.

Purpose
-------

The script's primary purpose is to traverse a specified directory, identify all JS and CSS files, and minify them to reduce their size, which is beneficial for improving load times and reducing bandwidth usage in web applications.

The script
----------

```js
const { minify } = require('uglify-js');
const fs = require('fs');
const path = require('path');
const CleanCSS = require('clean-css');

function minifyJSFiles(directory) {
    fs.readdir(directory, (err, files) => {
        if (err) {
            console.error('Error reading directory:', err);
            return;
        }

        files.forEach(file => {
            const filePath = path.join(directory, file);
            fs.stat(filePath, (err, stats) => {
                if (err) {
                    console.error('Error retrieving file stats:', err);
                    return;
                }

                if (stats.isDirectory()) {
                    // Recursive call for directories
                    minifyJSFiles(filePath);
                } else if (path.extname(file) === '.js') {
                    fs.readFile(filePath, 'utf8', (err, data) => {
                        if (err) {
                            console.error('Error reading file:', err);
                            return;
                        }

                        const minifiedCode = minify(data).code;
                        const minifiedFilePath = filePath;

                        fs.writeFile(minifiedFilePath, minifiedCode, err => {
                            if (err) {
                                console.error('Error writing minified file:', err);
                                return;
                            }

                            console.log(`Minified ${filePath} to ${minifiedFilePath}`);
                        });
                    });
                }
            });
        });
    });
}


function minifyCSSFiles(directory) {
    fs.readdir(directory, (err, files) => {
        if (err) {
            console.error('Error reading directory:', err);
            return;
        }

        files.forEach(file => {
            const filePath = path.join(directory, file);
            fs.stat(filePath, (err, stats) => {
                if (err) {
                    console.error('Error retrieving file stats:', err);
                    return;
                }

                if (stats.isDirectory()) {
                    // Recursive call for directories
                    minifyCSSFiles(filePath);
                } else if (path.extname(file) === '.css') {
                    fs.readFile(filePath, 'utf8', (err, data) => {
                        if (err) {
                            console.error('Error reading file:', err);
                            return;
                        }

                        const minifiedCode = new CleanCSS().minify(data).styles;
                        const minifiedFilePath = filePath;

                        fs.writeFile(minifiedFilePath, minifiedCode, err => {
                            if (err) {
                                console.error('Error writing minified file:', err);
                                return;
                            }

                            console.log(`Minified ${filePath} to ${minifiedFilePath}`);
                        });
                    });
                }
            });
        });
    });
}

// Example usage:
const directoryPath = process.argv[2];
minifyCSSFiles(directoryPath);
minifyJSFiles(directoryPath);
```

Components
----------

The script is divided into two main functions:

-   `minifyJSFiles(directory)`: Scans the specified directory for JS files and minifies them.
-   `minifyCSSFiles(directory)`: Scans the specified directory for CSS files and minifies them.

Each function is designed to handle directory traversal, file identification, file reading, content minification, and overwriting the original files with the minified content.

### Detailed Function Descriptions

#### `minifyJSFiles(directory)`

1.  Directory Reading: Begins by reading the contents of the provided directory.
2.  Error Handling: If an error occurs during directory reading, it logs the error and exits the function.
3.  File Processing:
  -   For each file, it constructs the full path and retrieves file statistics to determine whether the item is a directory or a file.
  -   Recursively handles subdirectories.
  -   If a JS file is found, it reads the file, minifies its content using `uglify-js`, and writes the minified content back to the same file.

#### `minifyCSSFiles(directory)`

1.  Directory Reading: Similar to `minifyJSFiles`, it reads the directory's contents.
2.  File Processing:
  -   Handles directories recursively.
  -   For CSS files, it reads the file, minifies the content using `CleanCSS`, and writes the minified content back to the file.

### Error Handling

Both functions incorporate comprehensive error handling to manage issues like reading files, accessing non-existent directories, or writing to files. Errors are logged to the console.

Usage
-----

### Prerequisites

-   Node.js must be installed on your system.
-   `uglify-js` and `clean-css` must be installed via npm. You can install them using:

```bash
npm install uglify-js clean-css
```

### Running the Script

To run the script, use the following command in the terminal:

```bash
node [script_name.js] [directory_path]
```

Replace `[script_name.js]` with the name of your script file and `[directory_path]` with the path to the directory containing the JS and CSS files you want to minify.

### Example

```bash
node minifyFiles.js /path/to/your/directory
```

This will execute the minification functions on the specified directory, processing both JS and CSS files.

Diagrams
--------

For better understanding, here are diagrams representing the flow of each function:

### `minifyJSFiles(directory)`

### `minifyCSSFiles(directory)`

*Note: Replace `URL_to_MinifyJSFiles_Flowchart` and `URL_to_MinifyCSSFiles_Flowchart` with actual URLs to the diagrams.*

Conclusion
----------

This script automates the process of minifying JavaScript and CSS files in a specified directory. It is built with efficiency and error handling in mind, providing a robust tool for web developers to enhance their web application's performance.

