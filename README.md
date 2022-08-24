# Hook Cleaner Wasm
Hook Cleaner removes unwanted compiler-provided exports and functions from a wasm binary to make it (more) suitable for being used as a Hook. This fork compiles Hook Cleaner to wasm which allows you to use Hook Cleaner directly in browser.

## WIP
This project is still being debugged

## Requirements
* emscripten and its dependencies

## Build
```bash
emcc cleaner.c -o cleaner.js -sEXPORTED_RUNTIME_METHODS=ccall,cwrap,callMain,FS -sMODULARIZE=1 -sEXPORT_NAME=Cleaner -sENVIRONMENT='web' -sEXPORT_ES6=1 -sUSE_ES6_IMPORT_META=1 -sFORCE_FILESYSTEM -sINVOKE_RUN=0 -sALLOW_TABLE_GROWTH -sWASM=1 -sALLOW_MEMORY_GROWTH=1
```

## Use in your webpage
```ts
import Cleaner from './cleaner';

// initialize cleaner
const cleaner = await Cleaner({
  locateFile: (file: string) => {
    return file;
  },
});

// Add your file:File eg. from file input to virtual file system
const cleanFile = (file: File) => {
  const buffData = await file.arrayBuffer();
  // create folder where to put file
  wasm.FS.mkdir("files");
  // create file
  wasm.FS.createDataFile(
    "/files",
    file.name,
    new Uint8Array(buffData),
    true,
    true
  );
  // call the cleaner main function with filename
  const ok = await wasm.callMain([`/files/${file.name}`]);
  // cleaner overwrites the cleaned file, let's read it
  const newFileUArr = wasm.FS.readFile(`/files/${file.name}`);

  // lets remove the file from the file system
  wasm.FS.unlink(`/files/${file.name}`);
  // lets remove the directory
  wasm.FS.rmdir("files");

  // Return file content (UInt8Array)
  return newFileUArr
}
```
