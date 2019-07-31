# Table of Contents

   1. [Introduction](#introduction)
   2. [Top-Level Properties](#top-level-properties)
   3. [The "Project Attributes" Object](#project-attributes)
   4. [The "Files" Object](#files)
      1. [Shared Keys](#shared-keys)
      2. [Less](#less-keys)
      3. [Sass & SCSS](#sass-and-scss-keys)
      4. [Stylus](#stylus-keys)
   

------------------------



# Introduction

A CodeKit 3 configuration file stores information about a CodeKit Project, its settings, and all the files it contains. 

### Name & Location:
   1. The file must named `config.codekit3` or `.config.codekit3`.
   2. The file must be located in the root Project folder. (The folder added to CodeKit as a Project.)
   
   
### Format:
The config file is JSON.


### Optionality:
There are only two required fields, both of which are top-level properties: `settingsFileVersion` and `creatorBuild`. All other fields in the Config File are optional.


### Case Sensitivity:
Keys are **case-sensitive.** Values are not.


### Sorting:
To prevent extraneous diffs during Git commits, CodeKit alphabetically sorts all keys in the Config File. While this is not a requirement, it is handy for users.


### 64-Bit Integers:
**Certain numbers in the Config File MUST be parsed as 64-bit integers.** 

This is because several numbers are actually bitmask values. The specific numerical values that MUST be treated as 64-bit integers are called out in this documentation.

As of 2019, JavaScript cannot represent 64-bit integers natively. The `Number` type stops at 53-bits of precision because JavaScript is a god-awful language. Fortunately, JavaScript is introducing the `BigInt` type to solve this problem. It is not officially part of the spec right now.

See https://golb.hplar.ch/2019/01/js-bigint-json.html for a workaround.

If you are **writing** a Config File from scratch and the values you intend to store for the properites that MUST be 64-bit integers do not exceed 53 bits of precision, you can safely write the value as a standard `Number`. For example, `fileType` can be set to `2 (2^1)` to represent a `Sass` file or `4096 (2^12)` to represent a `Markdown` file. 

If you are **reading** a Config File, you must treat numbers as 64-bit integers because you do not control whether or not they have bits beyond (2^53) "enabled". It is NOT safe to use `Number` to represent numberical values in this case.

-----------------------------


# Top Level Properties

The top level of the Config File is an object of type `{"key": value}`. It looks like this:

```json
{
    "AAInfo": "A string generated by CodeKit explaining the purpose of this file to the user",
    
    "buildSteps": [],
    
    "creatorBuild": "31491",
    
    "files": {},
    
    "hooks": [],
    
    "manualImportLinks": {},
    
    "projectAttributes": {},
    
    "projectSettings": {},
    
    "settingsFileVersion": "3"
}
```

### "AAInfo" — `String`
Anything you provide will be overwritten by CodeKit, but you may want to explain that this is a CodeKit configuration file and the app can be downloaded at https://codekitapp.com/

### "buildSteps" - `[{"key": value, ...}, ...]`
This stores information about each Build Step that a user has defined. It is an `Array` of `Objects` of type `{"key" : value}`. [See Details](#build-steps)

### "creatorBuild" - `String` (REQUIRED)
This is the build number for the version of CodeKit that created this file. The number is visible in CodeKit's About window. You MUST supply a **string** that contains a number equal to the version of CodeKit you're targeting. CodeKit will ignore any Config File that is missing this value.

As of August 2019, use `"31491"`, which is CodeKit 3.9.2. This documentation was written against that CodeKit version.

Whatever build number you enter here is the minimum version of CodeKit that your users should have. If your users have an older version of CodeKit, the Config File will still be read and any values that apply in the older version of the app will be used. However, any values that don't apply to the older version of CodeKit (for a feature that does not exist in the older version) will be discarded permanently.

You are guaranteed forward-compatibility. Your Config File will work in any future release of CodeKit and will be automatically migrated to future formats, if needed.

### "files" - `{"key": {"key": value, ...}, ...}`

This is an `Object` that stores settings for each individual file in the Project. [See Details](#files).

### "hooks" - `[{"key": value, ...}, ...]`

This is an `Array` of `Objects` of type `{"key": value}` that store information about each [Hook](https:/codekitapp.com/help/hooks/) that is defined for this Project. [See Details](#hooks).

### "manualImportLinks"

This stores information about import links between files that the user has created via drag-and-drop in CodeKit's UI. When parsing a Config File, you should treat this value as opaque and write it, unmodified, to your output. When creating a Config File from scratch, you should omit this property.

### "projectAttributes" — `{"key": value, ... }`

This is an `Object` that stores basic biographical information about a Project. [See Details](#project-attributes)

### "projectSettings" - `{"key": value, ... }`

This is an `Object` that stores the Project Settings for this Project. It contains configuration for tooling such as ESLint, Babel, Autoprefixer, etc. It also provides "default" values to fall back on when a file in `files` (above) is missing a value for a particular setting. If that value is *also* missing from `projectSettings`, CodeKit falls back to the "Defaults for New Projects" that the user has specified. At that level, **every** possible setting is guaranteed to have a value. [See Details](#project-settings)

### "settingsFileVersion" — `String` (REQUIRED)

Pass the `String` `"3"` for this value. CodeKit will ignore any Config File that is missing this value.


----------------------------

# Project Attributes

This top-level property is an `Object` that contains basic biographical information about a Project. It looks like this:

```json
"projectAttributes": {
   "creationDate": 586072330.73522604,
   "displayValue": "The Project Name As Shown in CodeKit's UI",
   "displayValueWasSetByUser: 0,
   "iconImageName": "brackets_purple",
   "iconImageWasSetByUser": 0
}
```

### "creationDate" — `Double`

The date the project was created. Stored as the number of seconds since `00:00:00 UTC on 1 January 2001` in `Double` format. If you omit this property, CodeKit will set it to whatever date the user first added this Project to the app, which is probably what you want.

### "displayValue" — `String`

The name of the Project as it appears in the UI. If this is omitted, the name of the Project Root Folder will be used.

### "displayValueWasSetByUser" - `Integer`

If `0`, CodeKit will keep `displayValue` in sync with the Project Root Folder name (as the user renames the folder, the `displayValue` will be updated to match). If `1`, renaming the folder has no effect on `displayValue`. If you omit this value, it defaults to `0`.

### "iconImageName" — `String`

This is *either* the name of a default Project Icon Image from the list below **OR** a relative path from the Project Root Folder to the image (JPG or PNG) that should be used as the Project Icon. If you supply a relative path, it **MUST** begin with a `/`.

If the specified image does not exist or you omit this property, CodeKit randomly selects a default icon. NOTE: If you want to maintain an exact icon, be sure to set `iconImageWasSetByUser` to `1`.

The available Project Icon Names that are bundled into CodeKit are:

* brackets_brown
* brackets_gray
* brackets_orange
* brackets_green
* brackets_magenta
* brackets_purple
* brackets_teal
* brackets-brick
* brackets-forest
* brackets-koamaru
* brackets-pink
* brackets-red
* brackets-slime
* brackets-ucla
* brackets-unblue
* brackets-glace
* brackets-pale
* brackets-payne
* brackets-silver
* brackets-tar
* brackets-azure
* brackets-cafe
* brackets-peach
* brackets-verd
* brackets-rose
* logo-wordpress
* logo-react
* logo-vue
* logo-angular
* meme-harold
* meme-putin
* meme-morpheus
* meme-thinking
* meme-ariel
* meme-ceiling
* meme-cocaine
* meme-facepalm
* meme-fry
* meme-girlfriend
* meme-grumpy
* meme-karate
* meme-lotr
* meme-soon
* meme-trap
* meme-wonka
* meme-kanye
* meme-hart
* meme-owl
* meme-punDog
* meme-angryPicard
* meme-cryingJordan
* meme-kahn
* meme-ive
* meme-dunnoGirl
* meme-greg
* meme-chloe
* meme-leo
* meme-drunkArcher
* meme-daenerys
* meme-gavin
* meme-krieger
* meme-archer
* meme-obama
* meme-maroney
* meme-crying
* meme-confession
* meme-doge
* meme-jake
* meme-ron
* meme-confusedBaby
* meme-angryBaby
* meme-yesBaby

### "iconImageWasSetByUser" — `Integer`

If `0`, CodeKit will automatically set an `apple-touch-icon.png` or `favicon` image as the Project Icon whenever it finds one anywhere in the Project. If `1`, `iconImageName` will not be changed when the app finds one of these files. If you omit this value, it defaults to `0`.



---------------------------

# Files

#### Discussion
This top-level property contains settings for each individual file in the Project. In CodeKit, every file has its own settings, which provides flexibility—one file can be set to minify while another does not, etc. 

When a new file is added to a Project for the first time, it won't have an entry in this object. Its initial settings are populated from whichever `languageDefaults` property of `projectSettings` is appropriate (based on file type). If `projectSettings` is missing an entry for a given `languageDefaults`, CodeKit falls back to the settings the user has specified under "Defaults For New Projects" within the app. At that level, **every** setting is guaranteed to have a value.

When creating a Config File, you may find it faster to specify *only* the file-by-file settings that differ from the `projectSettings` you have specified.

When *parsing* a Config File, you should never discard information from this object, as the user may have customized settings on a file-by-file basis and those customiziations would be lost (and restored to Project Settings values) if you discarded this object.

#### The Files Object

The files object is keyed by the **relative** path from the Project Root Folder to each file. The path **MUST** begin with a `/`. The exact keys in each file's object depend on the type of file. Here is an example for an `SCSS` file:

```json
"files": {
    "/subfolder/file.scss": {
		"aP": 0,
		"bl": 0,
		"dP": 5,
		"dS": 0,
		"ft": 4,
		"ma": 0,
		"oA": 2,
		"oAP": "/build/subfolder/file.css",
		"oF": 0,
		"oS": 0,
		"uL": 1
		},
    "/someOtherFile.js": {
      ...
    }
}
```

### Abbreviated Keys
The keys are abbreviated in each `file` object because in Projects with tens of thousands of files, the extra characters add megabytes to the size of the Config File. 


## Shared Keys
All files have the following keys, regardless of their type:


### "ft" — `Integer`

This stands for "File Type". It is one of the `Integer` values from the table below. CodeKit uses this (along with other heuristics) to determine the "absolute truth" about a file's type. (A file's extension alone is not a reliable indicator because, for example, `*.js` and `*.jsm` and `*.es6` can all mean "JavaScript".)

**CRITICAL:** If you are parsing the Config File in JavaScript, you must use `BigInt` to represent this value, as it may contain a value that is larger than `2^53`. If you are writing a Config File from scratch, you must use `BigInt` to represent this number for all values larger than `2^53`


| File Type             |  Extension | Value | Raw Value |
| --------------------- | ---------- | ----- | --------- |
| Less                  | .less      | 2^0   | 1         |
| Sass                  | .sass      | 2^1   | 2         |
| SCSS                  | .scss      | 2^2   | 4         |
| Stylus                | .styl      | 2^3   | 8         |
| CSS                   | .css       | 2^4   | 16        |
| CoffeeScript          | .coffee    | 2^5   | 32        |
| Literate CoffeeScript | .litcoffee | 2^18  | 262144    |
| JavaScript            | .js        | 2^6   | 64        | 
| TypeScript            | .ts        | 2^7   | 128       |
| Haml                  | .haml      | 2^8   | 256       |
| Pug                   | .pug       | 2^9   | 512       |
| Slim                  | .slim      | 2^10  | 1024      |
| Kit                   | .kit       | 2^11  | 2048      |
| Markdown              | .md, .mmd  | 2^12  | 4096      |
| Other*                | [N/A]      | 2^13  | 8192      |
| JPEG                  | .jpg, .jpeg| 2^14  | 16384     |
| PNG                   | .png       | 2^15  | 32768     |
| SVG                   | .svg       | 2^21  | 2097152   |
| GIF                   | .gif       | 2^22  | 4194304   |
| JSON                  | .json      | 2^19  | 524288    |
| Folder (Directory)    | [N/A]      | 2^16  | 65536     |


#### The "Other" File Type:
This is used for any type of file that is not otherwise listed above. It represents files for which CodeKit has no specific built-in processing. Examples: `.php`, `.html`, `.xml`, `.rb`, etc.

#### Sass vs SCSS:
Note that CodeKit uses two different `ft` values to represent `*.scss` and `*.sass` files. This is done so that the app is future-proof in case Sass and SCSS files ever adopt different, exclusive options.

#### The "Folder" Type:
Users can tell CodeKit to skip indexing a particular folder. They do this when the folder contains too many items that aren't relevant and indexing it would take a long time (e.g. `node_modules` folders). When the user does this, CodeKit does not store settings for each file in the folder, nor does it display those files in the UI. Instead, it asks the user what should be done with this folder (and its contents) when the Project builds—should the folder be ignored, or should it be copied into the build directory? To store this setting, an entry in the `files` object may have the value `65536` for `ft`, which means it refers to a folder instead of a file.

#### Reserved Values:
Do not use any value that is not explicitly listed in the table above. Doing so will cause CodeKit to ignore the entire settings object for that particular file.



### "oA" — `Integer`

This stands for "Output Action". It specifies what CodeKit will do when a given file changes on disk or is processed during a Build. The value is one of:

| Value  | Action          | Details |
| ------ | --------------- | ------- |
| 0      | Compile/Process | CodeKit will process this file on change and during Builds. If the file is of a type for which CodeKit does not have built-in processing AND the user has not specified a Hook to handle this type of file, the file will be copied to its output path as-is.
| 1      | Ignore          | On file-change and during Builds, this file will not be processed, but any files that import it will be.
| 2      | Copy            | On file-change and during Builds, CodeKit will copy this file as-is to its Output Path, **without** any processing/compiling.



### "oAP" — `String`

This stands for "Output Abbreviated Path". It is the relative path from the Project Root Folder to the Output File you want to create when the given file is processed. It **MUST** begin with a `/`.

Note: CodeKit *does* allow setting an Output Path to a location outside of the Project Root Folder, but that is extremely discouraged. If the `OutputPathIsOutsideProject` flag is set, CodeKit will interpret the path provided here as relative to the disk root instead of the Project Root Folder.


### "oF" - `String`

This stands for "Output Flags". It is a **bitmask** (a single number where the value of each bit represents a boolean). The default value is `0`. Currently, only the first three bits are used:


| Bit Position    | Name                       | Explanation     |
| --------------- | -------------------------- | --------------- |
| 0               | OutputActionWasSetByUser   | CodeKit automatically changes a file's Output Action to `ignore` when that file is imported into others. If the value of this bit is `1`, the app will not automatically change the Output Action. CodeKit sets this bit to `1` anytime the user explicitly chooses an Output Action for the file in the UI.
| 1               | OutputPathWasSetByUser     | CodeKit adjusts a file's Output Path automatically when the file is renamed, moved, or when the Build folder is enabled/disabled. If this bit is set to `1`, the Output Path is locked; it will not be automatically adjusted. This bit is set to `1` when a user explicitly sets an Output Path for a file or clicks the "lock" button in CodeKit's UI.
| 2               | OutputPathIsOutsideProject | If this bit is set to `1`, CodeKit interprets the path supplied for the `oAP` property as relative to the disk root instead of the Project Root Folder.


## Less Keys

In addition to the [shared keys](#shared-keys) common to all files, Less files have these keys:


### "aP" — `Integer`

`Autoprefixer` If this value is `1`, CodeKit will run Autoprefixer on the compiled CSS file. If the value is `0`, CodeKit will *not* run Autoprefixer.


### "bl" - `Integer`

`Bless` If this value is `1`, CodeKit will run Bless on the compiled CSS file. If the value is `0`, CodeKit will *not* run Bless.


### "eJ" — `Integer`

`Enable JavaScript` If this value is `1`, CodeKit will set the Less compiler's "Enable JavaScript" option to true. If the value is `0`, that option will be set to false.


### "ie" - `Integer`

`IE Compatibility` If this value is `1` CodeKit will enable the "IE Compatibility" option on the Less compiler. If it is `0`, that option will be set to false.


### "iI" — `Integer`

`Insecure Imports` If this value is `1` CodeKit will enable the "Allow Insecure Imports" option on the Less compiler. If it is `0`, that option will be set to false.


### "ma" — `Integer`

`Source Map` If this value is `1` CodeKit will create a source map when compiling this file. If the value is `0`, it will *not* create a source map.


### "mS" - `Integer`

`Math Style` This value controls which "Math Style" CodeKit enables for the Less compiler, which defines when the compiler performs math operations (see the Less docs for details). It is one of these values:

| Value             |  Math Style                             |
| ----------------- | --------------------------------------- |
| 0                 | Always Except Division Requires Parens  |
| 1                 | Always                                  |
| 2                 | Only Inside Parens                      |
| 3                 | Strict-Legacy (Deprecated)              |


### "oS" - `Integer`

`Output Style` This controls which output style the Less compiler uses. It is one of these values:

| Value             |  Output Style         |
| ----------------- | --------------------- |
| 0                 | Regular               |
| 1                 | Compressed (Minified) |


### "rwU" — `Integer`

`Rewrite URLs` This value determines which setting CodeKit passes to the Less compiler for the "rewrite URLs" option. It is one of these values:

| Value             |  Output Style          |
| ----------------- | ---------------------- |
| 0                 | Do not rewrite URLs    |
| 1                 | Rewrite ALL URLs       |
| 2                 | Rewrite only local URLs|


### "sI" - `Integer`

`Strict Imports` If this value is `1`, CodeKit will enable the "strict imports" Less compiler option when compiling this file. If the value is `0`, this compiler option is set to false.


### "sU" — `Integer`

`Strict Units` If this value is `1`, CodeKit will enable the "strict units" Less compiler option when compiling this file. If the value is `0`, this compiler option is set to false.



## Sass and SCSS Keys

In addition to the [shared keys](#shared-keys) common to all files, Sass and SCSS files have these keys:


### "aP" — `Integer`

`Autoprefixer` If this value is `1`, CodeKit will run Autoprefixer on the compiled CSS file. If the value is `0`, CodeKit will *not* run Autoprefixer.


### "bl" - `Integer`

`Bless` If this value is `1`, CodeKit will run Bless on the compiled CSS file. If the value is `0`, CodeKit will *not* run Bless.

### "dP" - `Integer`

`Decimal Precision` This is the value passed to the Sass compiler for the "Decimal Precision" option. It should be between `0` and `10`. 


### "dS" - `Integer`

`Debug Style` This value controls which "Debug Style" CodeKit enables for the Sass compiler. It is one of these values:

| Value             |  Debug Style                            |
| ----------------- | --------------------------------------- |
| 0                 | None                                    |
| 1                 | Print line numbers above selectors      |
| 2                 | Print full debug info                   |


### "ma" — `Integer`

`Source Map` If this value is `1` CodeKit will create a source map when compiling this file. If the value is `0`, it will *not* create a source map.


### "oS" - `Integer`

`Output Style` This controls which output style the Sass compiler uses. It is one of these values:

| Value             |  Output Style         |
| ----------------- | --------------------- |
| 0                 | Nested                |
| 1                 | Expanded              |
| 2                 | Compact               |
| 3                 | Compressed            |


### "uL" — `Integer`

`Use Libsass` If this value is `1`, CodeKit will use the Libsass compiler. If it is `0`, CodeKit will use the legacy Sass compiler. (Libsass is the default and is highly recommended.)


## Stylus Keys

In addition to the [shared keys](#shared-keys) common to all files, Stylus files have these keys:







----------------------------


# Build Steps
This is the top-level property named `buildSteps`, which is an `Array` of `{"string": value}`. There are currently two kinds of Build Steps: "run a script", "process certain files", and "process remaining files". The `buildSteps` Array may contain any number of the first two types and **exactly one** of the third type. The order matters; it is the order in which the user expects the steps to execute. Each type of Build Step has a slightly different structure:

### Process Certain Files








    
    
    
