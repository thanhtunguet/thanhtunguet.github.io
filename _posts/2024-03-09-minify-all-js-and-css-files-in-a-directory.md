---
title: "Minify all JS and CSS files in a given directory, using Node.js"
date: 2024-03-09 00:01:00 +0700
categories: [Programming, Javascript]
tags: [Programming, Javascript, Scripts]
pin: false
---

## Introduction

In today's web development landscape, optimizing the performance of our applications has become paramount. One of the most effective techniques for reducing load times and improving site performance is by minifying JavaScript (JS) and Cascading Style Sheets (CSS) files. Minification involves removing unnecessary characters like white spaces, comments, and newlines from the source code, resulting in smaller file sizes without altering their functionality.

Node.js, with its robust ecosystem and vast array of libraries, provides developers with powerful tools to automate tasks and streamline workflows. In this blog post, we will explore how to leverage Node.js to minify all JS and CSS files within a specified directory. By harnessing the capabilities of Node.js, we can create a simple yet efficient solution to optimize our web assets effortlessly. So let's dive in and learn how to enhance the performance of our web applications with Node.js-powered minification.

## The script

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

## Usage

```bash
node script.js /path/to/folder/
```
