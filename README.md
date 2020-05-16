# LuaWAK

LuaWAK is a single-file LuaJIT library that makes use of FFI to read, alter and write files in the Noita WizardPak (.wak) format, such as the game's assets archive `data.wak`.

# Basic Usage

## Extracting files

```lua
local wak = require("wak") -- load the library
                           -- you can also use something like:
			   -- local wak = loadfile("mods/my_mod/files/wak.lua")()

local ar = wak.open("data/data.wak") -- open the .wak file

local file = ar:open("data/enemies_gfx/iceskull.png") -- open any file within the data.wak

file:write("iceskull.png") -- write the data into a file on-disk
```

## Reading files
```lua
local wak = require("wak")
local ar = wak.open("data/data.wak")
local materials_file = ar:open("data/materials.xml")

local materials_xml = materials_file:read() -- read the contents of the file into a string

print(materials_xml)

for file in ar:files() -- iterate all the files in the archive
    print(file)
end
```

## Modifying archives
```lua
local wak = require("wak")
local ar = wak.open("data/data.wak")

ar:set("data/materials.xml", "<Materials> </Materials>") -- set the contents of the file

ar:add("data/my_file.txt", "Hello!") -- add a new file

local my_file = ar:open("data/my_file.txt") 
print(my_file:read()) -- added files work like normal

ar:write("data/data.wak.new") -- write the modified archive
```

## Creating new archives 

```lua
local wak = require("wak")
local ar = wak.new()

ar:add("1.txt", "One") -- add a file
ar:add("2.txt", "Two") -- add another file
ar:set("3.txt", "Three") -- :set does the same thing

ar:write("my.wak") -- write the new archive to disk
```
# API

* global table `wak`
	* function `open(path : string, lenient : boolean?) : wak_archive`

      opens a .wak file and returns its representation as a `wak_archive` object  
      if the `lenient` optional argument is passed as `true` the function will return `nil` if the file can't be opened, otherwise an error will be raised

    * function `new() : wak_archive`

      creates a new in-memory wak archive

* type `wak_archive`
	* field `path`

      contains the path that the archive has been loaded from  
      initialized with `"(memory)"` for archives created with `wak.new()`
	* function `preload_all()`

      preloads metadata about all files  
      luawak conserves memory and maintains speed by avoiding reading the entire file - instead, chunks are loaded as they are needed, e.g. to grab a particular file; this function forces the entire file to be loaded (but note that the file stream will still be owned by the object)

	* function `open(path : string) : wak_file` 

      opens a file within the archive and returns it

	* function `extract(path : string, out_path : string)`

      extracts a file from the archive into an on-disk file  
      will do nothing if `path` doesn't exist in the archive

    * function `add(path : string, content : string) : wak_file`

      adds a file named `path` with `content` as the data to the archive  
      returns the `wak_file` object that was added  
      will raise an error if the file exists
      will cause the entire archive to be loaded into memory

    * function `set(path : string, content : string) : wak_file`

      adds or changes a file named `path` in the archive, setting its content to `content`  
      returns the `wak_file` object that was added  
      will work for both existing files and new files  
      will not load any more data from the archive than is already in memory

    * function `import(path : string, imp_path : string) : wak_file`

      adds or changes a file named `path` in the archive, setting its content to the data in a file specified by `imp_path`  
      same behavior as `:set`

    * function `remove(path : string)`

      removes the file at `path` from the archive  
      will do nothing if the file doesn't exist
    
    * function `count_files() : number`

      returns a count of all files in the archive, including files added by `:add`/`:set`/`:import`  
      will cause the entire archive to be loaded into memory

    * function `files() : iterator<wak_file>`

      the `:files` function acts as an iterator, providing a `wak_file` object each loop  
      it's used similar to `ipairs` and `pairs`: `for file in archive:files()`

    * function `force_rebuild_on_write()`

      by default, if no changes are made to an archive, `:write` is essentially just a copy of the file  
      if changes *are* made, the function automatically switches to rebuilding the entire archive from scratch  
      if this function is called once before any calls to `:write`, luawak will drop the above behavior and always choose to rebuild the archive

    * function `write(out_path : string)`

      writes the whole archive to a file on disk  
      if the archive has been modified or `:force_rebuild_on_write` has been called before, the archive will be rebuilt from scratch at the provided file path  
      this function will reject writing to an `out_path` that is the same as the path the archive had been loaded from due to luawak's streaming behavior - see `:write_and_dispose` below

    * function `write_and_dispose(out_path : string)`

      same behavior as `:write`  
      however, after this function quits, the object is disposed and unusable (see `:dispose`)  
      this method **can** write to the same `out_path` as the path that the archive was initially loaded from

	* function `dispose()`

      closes the underlying file stream and disposes of the archive's resources, rendering it unusable  
      note: `dispose()` is called automatically when the archive object is garbage collected; calling it more than one time is perfectly valid and has no effect  
      calling any functions on a disposed object will raise errors

* type `wak_file`
	* field `length`

      length of the file in bytes

	* field `path`

      path to the file within the .wak archive

	* function `read() : string`

      returns the data of the file as a string

	* function `write(out_path) nil`

      writes the wak file into an on-disk file at the path specified by `out_path`

# License

LuaWAK is licensed under the MIT license. See `LICENSE` in this repository.
