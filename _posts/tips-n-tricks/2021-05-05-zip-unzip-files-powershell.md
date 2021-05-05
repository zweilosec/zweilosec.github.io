---
description: >-
  How to Zip and Unzip Files Using the Compress-Archive and Expand-Archive PowerShell cmdlets for easier file exfiltration
title: How to Zip and Unzip Files Using PowerShell                    # Add title of the machine here
date: 2021-05-05 08:00:00 -0600                           # Change the date to match completion date
categories: [Tips & Tricks]                     # Categories
tags: [powershell, zip]     # TAG names should always be lowercase; add relevant tags
show_image_post: true                                    # Change this to true
#image: /assets/img/                # Add image here for post preview image
---

Recently, while working on the Hack the Box machine [Sharp](https://zweilosec.github.io/posts/sharp/) I encountered a situation where I needed to exfiltrate a whole directory full of files and sub-folders back to my machine.  Rather than trying to copy each file separately, or writing a script to recursively copy the files to my machine, I decided it would be much easier to zip the files up into one neat little package using PowerShell's built-in functionality. By using one simple PowerShell cmdlet you can compress an assortment of files into a smaller, more portable file that is much easier to exfiltrate off of a system. 

## How to Zip Files Using PowerShell

### The `Compress-Archive` cmdlet

```powershell
PS C:\Users\User> Get-Help Compress-Archive

NAME
Compress-Archive

SYNTAX
Compress-Archive [-Path] <string[]> [-DestinationPath] <string> [-CompressionLevel {Optimal | NoCompression |
Fastest}] [-WhatIf] [-Confirm]  [<CommonParameters>]

Compress-Archive [-Path] <string[]> [-DestinationPath] <string> -Update [-CompressionLevel {Optimal |
NoCompression | Fastest}] [-WhatIf] [-Confirm]  [<CommonParameters>]

Compress-Archive [-Path] <string[]> [-DestinationPath] <string> -Force [-CompressionLevel {Optimal | NoCompression
| Fastest}] [-WhatIf] [-Confirm]  [<CommonParameters>]

Compress-Archive [-DestinationPath] <string> -LiteralPath <string[]> -Update [-CompressionLevel {Optimal |
NoCompression | Fastest}] [-WhatIf] [-Confirm]  [<CommonParameters>]

Compress-Archive [-DestinationPath] <string> -LiteralPath <string[]> -Force [-CompressionLevel {Optimal |
NoCompression | Fastest}] [-WhatIf] [-Confirm]  [<CommonParameters>]

Compress-Archive [-DestinationPath] <string> -LiteralPath <string[]> [-CompressionLevel {Optimal | NoCompression |
Fastest}] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

According to Microsoft's [documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.archive/compress-archive?view=powershell-7.1):

> The `Compress-Archive` cmdlet creates a compressed, or zipped, archive file from one or more specified files or directories. An archive packages multiple files, with optional compression, into a single zipped file for easier distribution and storage. An archive file can be compressed by using the compression algorithm specified by the `CompressionLevel` parameter.

Be aware that there is a limitation on the file size of archives created using this cmdlet.

> The `Compress-Archive` cmdlet uses the Microsoft .NET API `System.IO.Compression.ZipArchive` to compress files. The maximum file size is **2 GB** because there's a limitation of the underlying API.

The simplest method of using this cmdlet is to zip up all the files in a single directory.

```powershell
Compress-Archive -Path $in_path -DestinationPath $out_path\$out_file
```

From a PowerShell terminal zip up the files in the `$in_path` directory by specifying the `$out_path` and `$out_filename`.  PowerShell will take everything inside of the specified directory and compress it, subfolders and all.

Note: Quotations around the file path are **necessary** when the file path contains a space!

## How to unzip files using PowerShell

Once you copy the files to your local machine, you can use any tools or programs you have available to open the zip archive.  However, if you wish to stick with using PowerShell, or must use it to extract the files on the same remote system, it is useful to know how to use the `Expand-Archive` cmdlet.

### The `Expand-Archive` cmdlet

To recover the files from a zip archive specify the `$in_path` where the `$zip_file` is located and the `$out_path` you want the files extracted to.  If you leave out the `-DestinationPath` parameter, PowerShell extracts the zip into the present working directory.  PowerShell will create the folder structure inside the zip and place the contents inside. If the folder already exists in the destination, PowerShell will return an error when it tries to unzip the files. To bypass this error, you can force PowerShell to overwrite the data with by using the `-Force` parameter.

```powershell
Expand-Archive -Path $in_path/$zip_file -DestinationPath $out_path -Force
```
Warning: Using the `-Force` parameter will irrevocably replace currently existing files in the `$out_path` with no further warning.

## Advanced Usage

### Zipping multiple files

In addition to zipping all of the files in a single directory, you can also zip multiple individual files or even multiple entire directories into one file.  To do this, use a comma to separate the files and/or folders when specifying the `-Path` parameter.

```powershell
Compress-Archive -Path ~\.ssh,~\Documents -DestinationPath $out_path\$out_file
```

Note: For files/folders inside the present working directory you do not need to specify a path for input files. However, you may want to always specify full paths for consistency (escpecially when embedding inside of a script).  

```powershell
Compress-Archive -Path Music,Documents -DestinationPath $out_path\$out_file
```

Be aware, if you specify a folder that is empty it will not appear in your zip file!

### Using the wildcard character `*` to further control selection of files

The `Compress-Archive` cmdlet lets you use a wildcard character (`*`) to further control selection of files for compression. There are three main ways you can use the wildcard character (`*`).  

1. Exclude the containing folder, 
2. Compress only files inside the target directory, not folders (makes the selection non-recursive), or 
3. Choose all files that match specific criteria. 

### Zip all files without the containing folder

By adding an asterisk (`*`) to the end of the file path you can tell PowerShell to only compress the files/folders inside of a directory, excluding the folder itself.

```powershell
Compress-Archive -Path $in_path\* -DestinationPath $out_path\$out_file
```

### Zip only files in a folder (non-recursive)

If you want to zip only the files in a direcrtory, excluding subdirectories, you can use the star-dot-star (`*.*`) notation. 

```powershell
Compress-Archive -Path $directory\*.* -DestinationPath $out-path\$out_file
```

Note: This method is a hack that only works for files that have a file extension.  Windows generally requires extensions, so this may not be an issue most of the time.  However, be aware that files without an extension will be excluded (and folders with a `.` in the name will be included - with all files inside!).

### Zip all files that match specific criteria

In a scenario where you have a folder with a number of different filetypes but only want to zip the files of one type you can use the wildcard to substitute for the filename and then specify the `$extension` of the filetype you wish to archive.

```powershell
Compress-Archive -Path $in_path\*.$extension -DestinationPath $out_path\$out_file
```

Note: This method is non-recursive and will not include files inside subdirectories.


## Updating an existing zip archive

You can update an existing zipped file by adding the `-Update` parameter to the `Compress-Archive` cmdlet. This will allow you to replace files with newer ones of the same name, or add completely new files.  This parameter can be combined with any of the previous file/folder selection methods.

```powershell
Compress-Archive -Path $in_path -DestinationPath $out_path\$out_file -Update 
```

## References

* [https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.archive/compress-archive?view=powershell-7.1](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.archive/compress-archive?view=powershell-7.1)

If you have comments, issues, or other feedback, or have any other fun or useful tips or tricks to share, feel free to contact me on Github at [https://github.com/zweilosec](https://github.com/zweilosec) or in the comments below!

If you like this content and would like to see more, please consider buying me a coffee! <a href="https://www.buymeacoffee.com/zweilosec"><img src="https://img.buymeacoffee.com/button-api/?text=Buy me a coffee&emoji=&slug=zweilosec&button_colour=FFDD00&font_colour=000000&font_family=Lato&outline_colour=000000&coffee_colour=ffffff"></a>
