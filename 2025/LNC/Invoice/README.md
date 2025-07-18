## Invoice
Author: johnd  
Challenge prompt: Do not enable macro! password: infected  
Challenges are available at https://github.com/Lag-and-Crash/2025-public/tree/main/challenges/forensics  

To begin, we unzip the file using 7z due to previously encountering an error.  
`
7z x invoice.zip
`  
Then, we input the provided password "infected" to unzip it. This gives us invoice.xlsm.  
I proceeded to run binwalk on it to view its contents and found something interesting:  
`
1875          0x753           Zip archive data, at least v2.0 to extract, compressed size: 7911, uncompressed size: 21504, name: xl\vbaProject.bin
`   
Looking back at the challenge prompt, we can see 'macro' be mentioned. This caught my eye as in Excel, a macro is a recorded sequence of actions or commands   
that can be replayed to automate repetitive tasks. More importantly, they are written in VBA.  
I then extracted the files using  
`
binwalk -e invoice.xlsm
`  
