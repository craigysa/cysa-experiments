# cysa-experiments <!-- omit in toc -->

* [Experimenting with .gitattributes UTF-16](#experimenting-with-gitattributes-utf-16)
  * [Scenarios](#scenarios)
  * [Tabulated Results](#tabulated-results)
  * [Step-by-step](#step-by-step)
  * [Validating experiment](#validating-experiment)

## Experimenting with .gitattributes UTF-16

Files encoded as utf-16 are treated as binary in github due to the presence of the BOM<sup>[1](#foot1)</sup>.
This prevents comparison in pull requests.
Can `working-tree-encoding` be used to resolve the problem?

See: <https://git-scm.com/docs/gitattributes#_working_tree_encoding>

### Scenarios

1. File need not be utf-16 but sometime are: (Visual Studio config may use utf-16 for new files)
   1. Ideally we should ensure utf-16 is not used for such files bc the encoding doesn't have universal support.
   2. So try forcing utf-8 in working tree (and utf-8 would also be used in repo) 🤞
   3. Otherwise force utf-16
   4. Must test permutations:
      1. Existing files: ANSI & utf-16 (before any setting change)
      2. Set `<file-pattern> text working-tree-encoding=UTF-8`
      3. Update existing files and add new test files
      4. If utf-8 fails, also test: `<file-pattern> text working-tree-encoding=UTF-16`
   5. In each permutation validate local file format (in binary) and test github comparisons.
2. File must be utf-16: (some .rc files in MFC dev)
   1. In this case utf-16 is preferred, and has been shown to work.
   2. Test as per point 1 to confirm any peculiarities
3. Finally, the `working-tree-encoding` feature is relatively new, test impact of git clients that do not support it.
   1. Working tree uses utf-8
      1. Presumably client gets utf-8 from repo (internal and intended format)
      2. Client commits utf-8
      3. Presumably no impact on repo
   2. Working tree uses utf-16
      1. Presumably client gets utf-8 from repo (internal format, and client is unaware to change)
      2. **NOTE:** *Client may have errors attempting to use file of incorrect format (e.g. compiling with utf-8 `.rc` may fail)*
      3. Client commits utf-8
      4. Presumably no impact on repo

### Tabulated Results

TBC: Annotation/legend for results. Use `Alt+Shift+F` to reformat.

| File\Mode    | Default | utf-8 | utf-16 |
| :----------- | :-----: | :---: | :----: |
| First utf-8  |   BIN   |   ?   |   ?    |
| First utf-16 |  text   |   ?   |   ?    |

### Step-by-step

* [x] Create Repo
* [x] Create pre-existing files (notepad on Windows will suffice, choose encoding when saving)
* [x] Verify binary structure. [Step 1 Output](#out-1)
* [x] Commit and push to github
* [x] Verify github handling of the files? _utf-16 is BIN, utf-8(&/BOM) is text_

<a name="out-1">1</a>Step 1 files show the exact hex content of each file prior to commit. Note also that line ends use Windows convention of CRLF.

```ps1
PS > ls .\step1.* | %{(cat $_.FullName -encoding byte|fhx);$_.Name,""}
           Path:

           00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F

00000000   FE FF 00 76 00 31 00 0D 00 0A                    þ..v.1....
step1.utf16be

00000000   FF FE 76 00 31 00 0D 00 0A 00                    .þv.1.....
step1.utf16le

00000000   76 31 0D 0A                                      v1..
step1.utf8

00000000   EF BB BF 76 31 0D 0A                             ï»¿v1..
step1.utf8bom
```

### Validating experiment

Need to check the actual bytes in the file to confirm whether it is utf-16. Editors that support utf-16 will hide the BOM<sup>[1](#foot1)</sup>.

<a name="foot1">1</a> BOM: Byte Order Marker. An optional sequence of bytes at the start of the text file intended to unambiguously describe the encoding. If included, it is  not considered part of the file. But, the bytes may cause non BOM-aware tools to mishandle the file.

END
