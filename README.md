# LuaWAK

LuaJIT library using FFI to read .wak files (like the Noita `data.wak`).

# Basic Usage

## Extracting files

```lua
local wak = require("wak") -- load the library
                           -- you can also use something like:
			   -- local wak = loadfile("mods/my_mod/files/wak.lua")()

local ar = wak.open("data/data.wak") -- open the .wak file

local file = ar:open("data/enemies_gfx/iceskull.png") -- open any file within the data.wak

file:write_to("iceskull.png") -- write the data into a file on-disk
```

## Reading files
```lua
local wak = require("wak")
local ar = wak.open("data/data.wak")
local materials_file = ar:open("data/materials.xml")

local materials_xml = materials_file:read() -- read the contents of the file into a string

print(materials_xml)
```

# API

* global table `wak`
	* function `open(path) -> wak_archive` - opens a .wak file and returns its representation as a `wak_archive` object

* type `wak_archive`
	* field `path` - contains the path that the archive has been loaded from
	* function `preload_all() -> nil` - preloads metadata about all files
	* function `open(path) -> wak_file` - opens a file within the archive and returns it
	* function `extract(path, out_path) -> nil` - extracts a file from the archive onto an on-disk file, will do nothing if `path` doesn't exist in the archive
	* function `dispose() -> nil` - closes the underlying file stream and disposes of the archive's resources, rendering it unusable

  note: `dispose()` is called automatically when the archive object is garbage collected; calling it more than one time is perfectly valid and has no effect  

* type `wak_file`
	* field `length` - contains the length of the file in bytes
	* field `path` - contains the path to the file within the .wak archive
	* function `read() -> string` - returns the data of the file as a string
	* function `write_to(out_path) -> nil` - writes the wak file into an on-disk file at the path specified by `out_path`

  note: when the original archive is disposed, all files become unusable; do take care in ensuring the original `wak_archive` object is preserved somewhere

# License

MIT. See `LICENSE` in this repository.
