## Layer Cake  
Author: lyrisng  
Challenge prompt: Layer cake is so good. I have an mp3 file all about layer cake. Maybe you can find the flag there?    

We have been provided with the file:  
-layer cake.mp3  

At first glance, the file looks like a typical mp3 file.  
```
$ file "layer cake.mp3"  
layer cake.mp3: MPEG ADTS, layer III, v1, 80 kbps, 32 kHz, Stereo  
```
To reveal more information about it, we binwalk the file.
```
$ binwalk 'layer cake.mp3'  

DECIMAL       HEXADECIMAL     DESCRIPTION  
--------------------------------------------------------------------------------  
69            0x45            Zip archive data, at least v2.0 to extract, name: layers/docProps/  
147           0x93            Zip archive data, at least v2.0 to extract, name: layers/docProps/app.xml  
618           0x26A           Zip archive data, at least v2.0 to extract, name: layers/docProps/core.xml  
1081          0x439           Zip archive data, at least v2.0 to extract, name: layers/word/  
1155          0x483           Zip archive data, at least v2.0 to extract, name: layers/word/document.xml  
4151          0x1037          Zip archive data, at least v2.0 to extract, name: layers/word/fontTable.xml  
4805          0x12C5          Zip archive data, at least v2.0 to extract, name: layers/word/settings.xml  
5986          0x1762          Zip archive data, at least v2.0 to extract, name: layers/word/styles.xml  
10081         0x2761          Zip archive data, at least v2.0 to extract, name: layers/word/theme/  
10161         0x27B1          Zip archive data, at least v2.0 to extract, name: layers/word/theme/theme1.xml  
11962         0x2EBA          Zip archive data, at least v2.0 to extract, name: layers/word/webSettings.xml  
12425         0x3089          Zip archive data, at least v2.0 to extract, name: layers/word/_rels/  
12505         0x30D9          Zip archive data, at least v2.0 to extract, name: layers/word/_rels/document.xml.rels  
12855         0x3237          Zip archive data, at least v2.0 to extract, name: layers/[Content_Types].xml  
13299         0x33F3          Zip archive data, at least v2.0 to extract, name: layers/_rels/  
13374         0x343E          Zip archive data, at least v2.0 to extract, name: layers/_rels/.rels  
15253         0x3B95          End of Zip archive, footer length: 22  
```
It appears that the file has another embedded in it after all. We proceed to extract the zip file from the mp3.
```
$ binwalk -e 'layer cake.mp3'  

DECIMAL       HEXADECIMAL     DESCRIPTION  
--------------------------------------------------------------------------------  
69            0x45            Zip archive data, at least v2.0 to extract, name: layers/docProps/  
147           0x93            Zip archive data, at least v2.0 to extract, name: layers/docProps/app.xml  
618           0x26A           Zip archive data, at least v2.0 to extract, name: layers/docProps/core.xml  
1081          0x439           Zip archive data, at least v2.0 to extract, name: layers/word/  
1155          0x483           Zip archive data, at least v2.0 to extract, name: layers/word/document.xml  
4151          0x1037          Zip archive data, at least v2.0 to extract, name: layers/word/fontTable.xml  
4805          0x12C5          Zip archive data, at least v2.0 to extract, name: layers/word/settings.xml  
5986          0x1762          Zip archive data, at least v2.0 to extract, name: layers/word/styles.xml  
10081         0x2761          Zip archive data, at least v2.0 to extract, name: layers/word/theme/  
10161         0x27B1          Zip archive data, at least v2.0 to extract, name: layers/word/theme/theme1.xml  
11962         0x2EBA          Zip archive data, at least v2.0 to extract, name: layers/word/webSettings.xml  
12425         0x3089          Zip archive data, at least v2.0 to extract, name: layers/word/_rels/  
12505         0x30D9          Zip archive data, at least v2.0 to extract, name: layers/word/_rels/document.xml.rels  
12855         0x3237          Zip archive data, at least v2.0 to extract, name: layers/[Content_Types].xml  
13299         0x33F3          Zip archive data, at least v2.0 to extract, name: layers/_rels/  
13374         0x343E          Zip archive data, at least v2.0 to extract, name: layers/_rels/.rels  

WARNING: One or more files failed to extract: either no utility was found or it's unimplemented  

$ ls -la                                                                                             
total 80  
drwxrwxr-x  6 vexdru vexdru  4096 Jun  2 10:08  .  
drwxr-xr-x 27 vexdru vexdru 12288 Jun  2 10:07  ..  
-rw-rw-r--  1 vexdru vexdru 15275 May 31 01:12 'layer cake.mp3'  
drwxrwxr-x  2 vexdru vexdru  4096 Jun  2 10:08 '_layer cake.mp3.extracted'  


$ cd '_layer cake.mp3.extracted'                                                                   
```
We then navigate to the extracted contents and continue to extract the word document files from inside. 
```
$ ls -la                                                                                             
total 24  
drwxrwxr-x 2 vexdru vexdru  4096 Jun  2 10:08 .  
drwxrwxr-x 6 vexdru vexdru  4096 Jun  2 10:08 ..  
-rw-rw-r-- 1 vexdru vexdru 15206 Jun  2 10:08 45.zip  

$ unzip 45.zip                                                                                       
Archive:  45.zip  
error [45.zip]:  missing 69 bytes in zipfile  
  (attempting to process anyway)  
error: invalid zip file with overlapped components (possible zip bomb)  
```
However, we encounter a problem unzipping the file. Hence we fix and merge it. 
```
$ zip -FF 45.zip --out cake_fixed.zip                                                                
Fix archive (-FF) - salvage what can  
 Found end record (EOCDR) - says expect single disk archive  
Scanning for entries...  
 copying: layers/docProps/  (0 bytes)  
 copying: layers/docProps/app.xml  (370 bytes)  
 copying: layers/docProps/core.xml  (361 bytes)  
 copying: layers/word/  (0 bytes)  
 copying: layers/word/document.xml  (2894 bytes)  
 copying: layers/word/fontTable.xml  (551 bytes)  
 copying: layers/word/settings.xml  (1079 bytes)  
 copying: layers/word/styles.xml  (3995 bytes)  
 copying: layers/word/theme/  (0 bytes)  
 copying: layers/word/theme/theme1.xml  (1695 bytes)  
 copying: layers/word/webSettings.xml  (358 bytes)  
 copying: layers/word/_rels/  (0 bytes)  
 copying: layers/word/_rels/document.xml.rels  (237 bytes)  
 copying: layers/[Content_Types].xml  (340 bytes)  
 copying: layers/_rels/  (0 bytes)  
 copying: layers/_rels/.rels  (233 bytes)  
Central Directory found...  
no local entry: layers/  
EOCDR found ( 1  15184)...  

$ ls -la                                                                                             
total 40  
drwxrwxr-x 2 vexdru vexdru  4096 Jun  2 10:09 .  
drwxrwxr-x 6 vexdru vexdru  4096 Jun  2 10:08 ..  
-rw-rw-r-- 1 vexdru vexdru 15206 Jun  2 10:08 45.zip  
-rw------- 1 vexdru vexdru 15129 Jun  2 10:09 cake_fixed.zip  
```
Unzip the fixed zipfile.  
```
$ unzip cake_fixed.zip  
Archive:  cake_fixed.zip  
   creating: layers/docProps/  
  inflating: layers/docProps/app.xml    
  inflating: layers/docProps/core.xml   
   creating: layers/word/  
  inflating: layers/word/document.xml    
  inflating: layers/word/fontTable.xml   
  inflating: layers/word/settings.xml   
  inflating: layers/word/styles.xml   
   creating: layers/word/theme/  
  inflating: layers/word/theme/theme1.xml   
  inflating: layers/word/webSettings.xml   
   creating: layers/word/_rels/  
  inflating: layers/word/_rels/document.xml.rels   
  inflating: layers/[Content_Types].xml   
   creating: layers/_rels/  
  inflating: layers/_rels/.rels       
```
Finally, we run grep to search for the flag and successfully obtain it.  
```
$ grep -aR "grey{"  
........paragraph" w:styleId="Heading5"><!-- grey{s0_f3w_lay3r5_w00p5} --><w:name w:val="heading.......  
```
Flag: grey{s0_f3w_lay3r5_w00p5}  
