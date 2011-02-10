sys    = require 'sys'
fs     = require 'fs'
{exec} = require 'child_process'

# ANSI Terminal Colors.
bold  = '\033[0;1m'
red   = '\033[0;31m'
green = '\033[0;32m'
reset = '\033[0m'

walk = (filename, callback) ->
  fs.stat filename, (err, stats) ->
    if stats.isFile()
      callback filename
    else if stats.isDirectory()
      fs.readdir filename, (err, files) ->
        walk "#{filename}/#{file}", callback for file in files

task 'build', 'copy all the JavaScript into the build directory', (options) ->
  exec "mkdir -p build && cp -r src/* build"

task 'clean', 'remove all the compiled JavaScript files', (options) ->
  exec "rm -rf build/*"

task 'min', 'minify the compiled JavaScript files', (options) ->
  jsmin = require('jsmin').jsmin
  
  walk 'build', (uncompressedFilename) ->
    if uncompressedFilename.indexOf(".min.") == -1
      fs.readFile uncompressedFilename, (err, uncompressedCode) ->
        minifiedFilename = uncompressedFilename.replace '.js', '.min.js'
        
        uncompressedCode = uncompressedCode.toString 'ascii'
        minifiedCode = jsmin uncompressedCode, 3
        
        console.log "Minifying #{red}#{uncompressedFilename}#{reset} to #{green}#{minifiedFilename}#{reset}."
        fs.writeFile minifiedFilename, minifiedCode

task 'lint', 'run JSLint on the compiled JavaScript files', (options) ->
  walk 'build', (file) ->
    if file.indexOf(".min.") == -1
      exec "jslint #{file}", (err, stdout, stderr) ->
        console.log "Running JSLint on #{green}#{file}#{reset}:"
        console.log stdout